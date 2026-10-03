# 按场景排障

[English](scenarios.en.md) · [首页](../README.zh-CN.md) · [部署](setup.zh-CN.md)

以下目标均为虚构示例，使用时换成私下取得的已授权地址。每次修改一个相关变量，重复同一项检查并记录结果。缺少 VPN 资源授权时，重装客户端无助于解决问题。

## 通用诊断

从本机入口逐层向目标检查：

| 层级 | 有用证据 | 可以证明什么 / 下一步 |
| --- | --- | --- |
| 运行时 | 容器状态、当前 Docker context、镜像架构 | VM 或容器未运行，属于本地前置条件缺失 |
| 监听端口 | Docker 端口映射、实际 loopback 端口的监听进程 | 有监听还需核对协议；HTTP 与 SOCKS 不能混用 |
| SOCKS | 显式使用 aTrust SOCKS5 的请求 | SOCKS 拒绝或代理认证错误发生在业务认证之前 |
| VPN | 支持的登录 UI、容器内到指定目标的路由 | 登录成功不能证明该目标已有资源授权和路由 |
| 目标 TCP | 经选定路径进行限时连接、SSH 或 TLS 握手 | 超时、拒绝、握手成功要分别记录 |
| HTTP/TLS | 状态码、证书校验、响应服务身份 | `401/403` 需要解释，代理或 SSO 网关也可能返回它 |
| 应用 | 真实页面、已认证 API 响应、Git/SSH 操作 | 这是用户需要的验收结果 |
| 持久运行 | 约定进程在有限范围重启后恢复，再测应用 | 容器恢复后仍可能需要重新登录/MFA |

以下命令不携带凭据。使用本机**实测**代理端口和已授权目标：

```sh
# 经 aTrust 访问内网 HTTP 服务，避免环境变量中的 bypass 干扰。
curl --noproxy '' --proxy socks5h://127.0.0.1:1080 \
  --connect-timeout 5 --max-time 15 -o /dev/null \
  -w 'http=%{http_code}\n' http://192.168.50.30:18080/v1/models

# 经可选的应用代理入口重复检查。
curl --noproxy '' --proxy http://127.0.0.1:7890 \
  --connect-timeout 5 --max-time 15 -o /dev/null \
  -w 'http=%{http_code}\n' http://192.168.50.30:18080/v1/models

# 经同一入口检查外网对照站点。
curl --noproxy '' --proxy http://127.0.0.1:7890 \
  --connect-timeout 5 --max-time 15 -o /dev/null \
  -w 'http=%{http_code}\n' https://example.com/
```

内网 API 自己返回 `401` 可以说明路径和 HTTP 服务可用，但尚未验证 Key、模型权限或推理。有网关时还要确认响应来自哪里。不要添加 `-k`、关闭严格 SSL 或安装未经核验的证书来获得“通过”。确需组织 CA 时，取得已核验的 CA，按组织允许的信任流程处理。

常见分界：

- **代理连接不上：**检查运行时、loopback 绑定、实际端口和协议，不猜另一个常用端口。
- **SOCKS 只通一个内网目标：**检查失败目标的容器路由、VPN 资源授权和防火墙/服务端口，不扫描整个内网。
- **curl 通、应用不通：**检查应用代理设置、bypass、扩展实际执行位置和证书存储。远程 SSH/容器扩展宿主里的 `127.0.0.1` 无法代表 Mac 上的监听端口。
- **agent 报 operation not permitted：**区分 harness sandbox 限制与真实网络故障，使用正式权限机制后按同样条件重测一次。
- **浏览器打开后登录跳转失败：**只提取跳转主机名和端口，继续按 OA 场景排查。

### DNS、网段冲突与 UDP

`socks5h://` 要求 SOCKS 代理解析主机名；curl 的 `socks5://` 在本地解析。仅公司 DNS 可解析的域名可能因此出现不同结果。HTTPS 保留原主机名，确保 Host/SNI 与证书校验成立；换成 IP 可能引入独立的 TLS 或虚拟主机错误。

`IP-CIDR,...,no-resolve` 规则不会为了匹配而强制解析域名，因此同时配置已核验域名与必要的 `/32` 地址规则。SSO 可能引入另一域名，观察到并确认路径后再添加。不要为了一个应用的解析问题开启第二套全局 DNS/TUN。

Mac 所在 LAN 与公司内网重叠时，分别比较宿主机与容器路由。容器内 VPN 路由正确时，显式 SOCKS 有时可以避开宿主机冲突；容器内部的路由冲突仍需处理。必要时向管理员索取不冲突的地址、路由或已授权跳板机。

示例验收范围是 **TCP** 应用。SOCKS UDP、QUIC、WebRTC 与系统 DNS 是独立能力。示例 SOCKS 节点关闭 UDP，不代表所有 UDP 包都会被丢弃、也不能排除它被其他策略处理。不能据此承诺全机防泄漏或 UDP 已覆盖。

## OA 与 SSO

一个“OA 网站”可能由多个服务组成。例如入口 `https://oa.corp.example` 跳到 `https://sso.corp.example`，再打开另一个主机或端口上的业务应用。

1. 在本地浏览器 Network 面板查看跳转链和失败请求，只记录协议、主机名、端口、状态码与资源类型。不分享含登录票据的完整 URL、查询 token、Cookie、授权头、响应正文或原始 HAR。
2. 判断每个必要目标应走内网 aTrust 还是现有外网策略；公共身份提供方不自动归入公司 VPN。
3. 在宽泛 bypass 规则前增加精确内网域名/IP 规则，同时检查 HTTP 与 WebSocket。保留浏览器现有可信证书及站点设置。
4. 在真实浏览器重新登录，验证落地页及用户需要的嵌入应用或文档流程。只打开登录页不足以通过验收。
5. 网络已通但反复跳登录时，继续核对会话过期、Cookie 限制、时钟偏差、SSO 回调注册和账号权限。增加路由不能修复无效会话。

域名规则不能补发 VPN 资源权限。只通过厂商 Web 网关开放的应用，应使用其文档规定的 URL 重写/访问方式，不虚构直接内网地址。

## 内网模型 provider

分清以下四项：

| 配置项 | 虚构示例 | 验证方法 |
| --- | --- | --- |
| 服务 base URL | `http://192.168.50.30:18080` | provider 文档和实际可达路径 |
| API 前缀 | `/v1` | 客户端是否自行追加，避免 `/v1/v1` |
| 模型 ID | `example-team/openai/example-chat` | 完整保留 provider 返回或文档说明的 ID |
| API Key | 在客户端私下输入 | 使用 SecretStorage、Keychain 或客户端支持的凭据存储 |

兼容 OpenAI 的客户端可能要求服务根地址，也可能要求 API 根地址，应先确认其约定。不能擅自把公司前缀模型 ID 换成猜测的公共模型名，也不能从名字推断上下文长度、工具或视觉能力。

验收顺序：

1. 先用上文不带认证的请求检查路径，只记录可公开的状态/错误类别。
2. 通过客户端安全输入框保存 Key，不放进 README、shell 参数/历史、进程命令行、截图、公开 issue 或受版本控制的配置。用户无需把 Key 发到 agent 聊天中。
3. provider 提供模型列表时先读取列表，再发一个最小已认证请求，例如“只回复 OK”；检查时关闭工作区/文件自动附带。
4. 在真实 IDE UI 中选择完整模型 ID，确认实际 provider/model 和完整响应。下拉列表刷新成功尚不够。
5. 有需求时验证流式响应并取消一次。区分客户端取消与已经证实的服务端取消或计费行为。工具、视觉和长上下文按需单独验收。

普通 JSON 成功而流式卡住时，检查代理缓冲、空闲超时、SSE/WebSocket 传输和客户端兼容。`404` 常涉及接口前缀或模型 ID，`401/403` 可能涉及 Key 权限或 provider 策略；保留真实且已脱敏的错误类别，不跨无关模型盲重试。

HTTP 不提供端到端 TLS；有条件时使用组织支持的 HTTPS 入口，VPN 本身不能证明应用层 TLS 已成立。不为公司 API 新建公开监听入口。

## GitLab 与 SSH

HTTPS Git 与 SSH Git 的代理机制不同。系统 HTTP 代理和 `ALL_PROXY` 不会自动配置 OpenSSH。

### HTTPS Git

先在浏览器验证 GitLab 页面与 SSO，再为一次只读 Git 操作显式指定代理：

```sh
git -c http.proxy=http://127.0.0.1:7890 \
  ls-remote https://gitlab.corp.example/team/demo.git
```

仓库路径是虚构示例。换成目标仓库，复用现有 credential helper，不在 URL 嵌入 token/密码。在 agent harness 中按其正式网络权限机制运行远端 Git，不把 sandbox 拒绝当作 GitLab 故障。

确需持久配置时，优先审查并设置仅该 GitLab 地址适用的 `http.<url>.proxy`，避免覆盖所有远端的全局代理。保存旧值，提供精确撤销命令；检查现有 helper、proxy 与 bypass 时不输出凭据。

### SSH Git 或其他内网 SSH 服务

通过已有可信渠道核对服务器主机键。宿主机直连路由确实可用时直接使用；否则显式使用实测 SOCKS5 入口。macOS `nc` 应先确认本机版本支持 `-X 5 -x`：

```sshconfig
# 合并到 ~/.ssh/config；以下均为示例值。
Host corp-gitlab
    HostName gitlab.corp.example
    User git
    Port 22
    IdentityFile ~/.ssh/id_ed25519_corp
    IdentitiesOnly yes
    ServerAliveInterval 30
    ServerAliveCountMax 3
    ProxyCommand /usr/bin/nc -X 5 -x 127.0.0.1:1080 %h %p
```

密钥文件必须实际存在且属于本机用户，不覆盖已有密钥、不复制他人私钥，保留其他 Host 配置。禁止关闭主机键校验；`ssh-keyscan` 本身不能证明服务器身份。还应确认所选代理命令能按预期解析内网主机名。

安装经独立核验的主机键、完成公钥授权后，执行 `ssh -o BatchMode=yes -T corp-gitlab`。GitLab 可能在认证成功后提示不提供交互 shell，并返回非零码；结合服务端消息判断，再用只读 Git 操作验证访问。不为连通性测试执行 push、切分支或改写历史。

## 外网模型目录

针对 VS Code 已有 Pro+、其他机器能用 Opus、本机却缺失 Claude 或显示 Upgrade 的情况，见[详细修复案例](copilot-claude.zh-CN.md)。其中包含故障观察、实际规则/版本变化、两项真实生成证据、逐步复测和[专项 agent prompt](../prompts/copilot-claude.zh-CN.md)。下文保留适用于其他公共模型服务的通用检查。

IDE 订阅和模型下拉列表需要分层检查：

1. 从服务支持的界面确认登录账号、套餐、组织策略及当前模型权限。可用性会变化，以当前官方文档为准。
2. 确认客户端版本及扩展兼容性，修改前记录版本。
3. 查看失败的模型目录/调用请求实际访问的目标，以及 Shadowrocket/Mihomo 命中策略。GitHub Copilot 可参考[官方网络指导](https://docs.github.com/en/copilot/how-tos/troubleshoot-copilot/troubleshoot-network-errors)，不能把记忆中的单个域名当作完整端点清单。
4. 如果已有 `DIRECT` 或 bypass 规则把必要的公共服务送到失败路径，窄范围修正为已有外网策略。没有路径依据时，不把公共服务送入公司 VPN。
5. 仅在需要时刷新认证/模型目录，再选择目标模型实际获得最小响应。保留设置和凭据，不删除整个 profile 或认证数据库。

源环境的相关维护中，路由变更和客户端更新/重启均发生在恢复之前。因此只能确认最终组合可用，不能推断所有 Claude 缺失都源于某一域名规则。自己的验收报告也应保留这种因果边界。

## 何时停止依赖动作并交接

缺少 VPN 资源授权、MFA、可信证书、账号权限或管理员提供的路由时，停止依赖这些条件的动作，完成仍可独立进行的本地检查。报告具体缺项及失败层级，使用[验收模板](acceptance-template.md)保存在本机私有目录。容器运行或 YAML 语法通过，均不能单独证明部署成功。
