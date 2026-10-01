# 布置与重启指南

[English](setup.en.md) · [原理](../README.zh-CN.md) · [场景排障](scenarios.zh-CN.md)

下列地址均为示例，先探测再替换。除明确写出 `docker exec` 的命令外，终端命令都在 Mac 上运行。不要把虚构内网地址当作扫描目标。

## 1. 发现已有环境

记录本机用户名、HOME、系统与 CPU 架构，读取当前容器运行时，不切换它的 context：

```sh
id -un
printf '%s\n' "$HOME"
uname -m
docker context show
docker ps --format '{{.Names}}\t{{.Image}}\t{{.Ports}}'
```

选定实际发现的容器，只读必要字段：

```sh
ATRUST_CONTAINER=your-existing-container
docker inspect "$ATRUST_CONTAINER" --format '{{.Config.Image}}'
docker port "$ATRUST_CONTAINER"
```

用 `docker image inspect <实际镜像> --format '{{.Os}}/{{.Architecture}}'` 核对安装的镜像架构，再与 `uname -m` 对照。上游 aTrust 有多架构版本，不要在 Apple Silicon 上无条件强制 amd64。只有镜像需要且运行时支持时才使用模拟执行。

在 Shadowrocket 中记录生效配置、路由模式、外网策略、本地 HTTP/SOCKS 监听，以及 LAN/TUN 绕过规则。只读所需字段，不整份打印容器环境变量、订阅、登录链接、私钥或应用数据库。

按实际端口检查监听，例如：

```sh
lsof -nP -iTCP:1080 -sTCP:LISTEN
lsof -nP -iTCP:1082 -sTCP:LISTEN
lsof -nP -iTCP:7890 -sTCP:LISTEN
```

端口有监听，只说明有进程绑定，不证明它说的是预期代理协议，也不证明 VPN 已认证。还要检查相关系统代理绕过列表：部分应用会先跳过私网 IP，代理内部的规则根本收不到请求。

通过 Shadowrocket 界面复制/导出生效配置作私有备份；将要修改的应用设置、Mihomo 文件同样先备份。备份放在 Git 之外；即便文件权限为 600，其中仍可能有订阅和凭据。

## 2. 复用 aTrust，仅在缺少时安装

### 已有环境

保留容器、镜像、挂载状态、门户和登录。会话过期时打开它支持的登录界面，仅补充缺失的门户、账号、MFA 或证书。服务器 SSH 密码不能当作 VPN 密码。

如果现有映射绑定 `0.0.0.0` 或 `::`，记录它可能向其他设备开放代理或登录界面。新建示例只绑定回环。修改已有 Docker 端口映射通常需要重建容器，应保留挂载和登录状态、说明中断影响，在连接正被使用时单独安排；不要为了照抄教程重建可用容器。

### 新机从零安装

选一种支持的容器运行时，并阅读[上游 aTrust 说明](https://github.com/docker-easyconnect/docker-easyconnect#readme)。本仓库 [Compose](../examples/compose.yaml) 只适用于 `hagb/docker-atrust`；其他镜像按自身文档配置。

1. 在仓库外建立私有目录：

   ```sh
   mkdir -p "$HOME/.config/atrust-inout" "$HOME/.local/share/atrust-inout/atrust"
   chmod 700 "$HOME/.config/atrust-inout" "$HOME/.local/share/atrust-inout/atrust"
   cp examples/atrust.env.example "$HOME/.config/atrust-inout/atrust.env"
   chmod 600 "$HOME/.config/atrust-inout/atrust.env"
   ```

2. 私下编辑该文件：填写受支持的平台、已存在的本机绝对状态目录、所选镜像、空闲端口和 VNC 登录密码。上游 VNC 密码选项最多八个字符，它与 VPN 账号密码不同。VPN 密码不进入 Compose 或命令参数。VNC 环境变量仍可被 Docker 管理者读取，回环监听也不隔离同机的其他用户或进程。
3. 通过镜像仓库和运行时确认镜像 tag、架构，缺少才拉取。验证后记录或固定可复现的镜像版本；模板中的 `latest` 只是起点。
4. 确认回环端口 `1080`、`5901` 与 aTrust web 登录回调端口 `54631` 空闲。回调映射遵循上游说明；另一套 aTrust 客户端可能占用同一端口。
5. 校验时不打印展开后的密码，然后启动：

   ```sh
   docker compose -p atrust-inout --env-file "$HOME/.config/atrust-inout/atrust.env" -f examples/compose.yaml config -q
   docker compose -p atrust-inout --env-file "$HOME/.config/atrust-inout/atrust.env" -f examples/compose.yaml up -d
   docker compose -p atrust-inout --env-file "$HOME/.config/atrust-inout/atrust.env" -f examples/compose.yaml ps
   ```

6. 用 VNC 客户端连接配置中的回环 VNC 端口，在 aTrust 界面输入管理员提供的门户与凭据，由用户完成 MFA。需要 web 登录时使用镜像支持的浏览器/回调流程。noVNC 或内置浏览器镜像是单独的选择，这份模板不默认启用它们。

模板按镜像要求提供 `NET_ADMIN` 和 `/dev/net/tun`，不使用 `--privileged` 或 host 网络。持久化 `/root` 含私有登录状态，不要为了排障直接删除。详见[上游 usage](https://github.com/docker-easyconnect/docker-easyconnect/blob/master/doc/usage.md)。

## 3. 先证明 VPN 路径，再修改应用分流

选择一个管理员已授权的目标。以下使用虚构模型服务：

```sh
ATRUST_SOCKS_PORT=1080
curl --noproxy '' --proxy "socks5h://127.0.0.1:${ATRUST_SOCKS_PORT}" \
  --connect-timeout 5 --max-time 15 -sS -o /dev/null -w 'HTTP=%{http_code}\n' \
  http://192.168.50.30:18080/v1/models
```

`--noproxy ''` 避免继承的 `NO_PROXY` 绕过这次探测。目标是域名时，`socks5h` 将解析交给 SOCKS 代理，不代表其他客户端也会这样解析。还要检查 curl 退出码。`401` 可能证明网络已通、进入认证边界；`404` 可能只是路径错误。二者都不能作为模型功能验收。

失败时对比容器与 Mac 的路由：

```sh
docker exec "$ATRUST_CONTAINER" ip route get 192.168.50.30
route -n get 192.168.50.30
```

不假定隧道接口一定叫 `utun7`。有路由也不证明目标接受连接。只有容器内存在路由时，SOCKS 路径仍可能可用，Mac 直连则不一定可用。VPN 未开放目标时，需要可路由地址、资源授权或已授权跳板；本地与远端同网段冲突也应明确处理，不靠反复重装客户端解决。

## 4. 选择改动最少的分流方式

### A. 只使用 Shadowrocket

1. 在应用内新增 SOCKS5 节点，指向**实测**的 aTrust 回环地址与端口。若本地 SOCKS 开启了认证，使用它自己的认证信息，不复用 VPN 密码。
2. 把节点或单成员策略命名为 `COMPANY-VPN`，唯一上游为 aTrust，不放入直连/外网回退。
3. 在当前配置的副本中增量合并[规则片段](../examples/shadowrocket.rules.example)，先替换虚构目标。精确域名与 `/32` 排在冲突的 LAN、GEOIP 或最终规则之前；保留外网策略。
4. 选择按配置分流，确认副本确实生效，通过规则测试/连接视图查看**实际命中**。
5. 测试真实浏览器页面和一个外网站点。若浏览器在进入代理前就绕过私网 IP，只调整相关绕过设置，或选择已验证的 TUN/应用路径；同时确认 TUN 没有重新捕获代理自己的上游流量。

不同版本的界面名称和规则能力可能变化，不直接修改运行中的 SQLite。规则保存到未激活的配置中不会生效。

### B. 可选 Mihomo 应用入口

当应用需要显式 HTTP 代理，或者它内部的请求实现没有使用现有系统链路时再增加。

1. 复用已有 Mihomo。缺少且已有 Homebrew 时，先检查 formula，再安装 `mihomo`；也可按[上游安装说明](https://wiki.metacubex.one/start/)操作。
2. 将[示例](../examples/mihomo.example.yaml)另存为**私有**配置，适配实际 aTrust SOCKS5 与 Shadowrocket HTTP 入口。任一代理需要认证时私下补充，不整份覆盖已有配置。
3. 用 `mihomo -t -f /absolute/private/path/config.yaml` 校验。它只证明配置可解析，不证明目标可达。
4. 首次前台验证可运行 `mihomo -f /absolute/private/path/config.yaml`，完成后只停止该测试进程。已有端口占用时不能再启动一个实例争抢。
5. 使用 Homebrew 后台服务时，先检查 `brew services info mihomo` 及它实际使用的配置路径，将增量修改应用到该文件，再仅启动/重启这个服务。不能假定后台服务会读取刚才前台命令指定的私有文件；启动后回读监听和日志。
6. 将 [VS Code 设置片段](../examples/vscode-settings.example.json)合并到正确的用户作用域，重载空闲窗口。其他应用按自身代理设置接入。证书校验保持开启；如需企业 CA，按组织支持的流程安装。

这个方案不需要第二个全局 TUN。示例不开放 controller API，也不设置全机系统代理。外网流量交给 Shadowrocket；它不能再把流量指回 Mihomo。DNS 与同网段冲突仍须单独诊断。规则语义参考 [Mihomo](https://wiki.metacubex.one/config/rules/)，应用限制参考 [VS Code 网络文档](https://code.visualstudio.com/docs/setup/network)。

## 5. 验收、重启与回滚

将[验收模板](acceptance-template.md)保存到仓库外，同时验证内网应用与外网应用。只保留简短、脱敏的证据，真实地址在本地私下核对。

启动顺序：容器运行时 → aTrust 容器 → 有效 VPN 会话 → Shadowrocket 配置/监听 → 可选 Mihomo → 应用。`restart: unless-stopped` 不负责启动 Mac 的运行时，也不能自动完成 MFA。休眠或重启后重新探测端点，没有实测就不承诺开机自启或自动恢复。

回滚按层进行：还原本次触达的应用代理字段，再还原增改的规则或切回备份配置，必要时停止本次新增的桥接服务。恢复原值，不删除用户无关设置。若本次新建了示例 stack，对应的 `docker compose ... down` 只用于停止/移除这套新容器；不用 `-v`，保留外部状态目录。不能把该命令套到用户原有 stack 上。
