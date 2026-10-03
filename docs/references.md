# References and boundaries / 参考与边界

Reviewed for this documentation on **2026-10-01**. Upstream behavior and client UI can change; verify the versions actually installed before deployment.

本教程于 **2026-10-01** 核对以下资料。上游行为和客户端界面可能变化，部署时以实际安装版本为准。

The Copilot case and its linked official guidance were additionally reviewed on **2026-10-03**; its successful generation records are dated **2026-10-01**, not a new live test.

Copilot 专项案例及链接的官方指导另于 **2026-10-03** 核对；成功生成证据发生于 **2026-10-01**，本次整理没有重新实测调用。

| Primary source / 一手来源 | Used for / 用途 |
| --- | --- |
| [docker-easyconnect README](https://github.com/docker-easyconnect/docker-easyconnect/blob/master/README.md) | aTrust image family, supported architectures and project scope / aTrust 镜像系列、架构与项目范围 |
| [docker-easyconnect usage](https://github.com/docker-easyconnect/docker-easyconnect/blob/master/doc/usage.md) | aTrust container capabilities, state, SOCKS/VNC/web callback ports and login options / 容器权限、状态目录、SOCKS/VNC/回调端口与登录选项 |
| [docker-easyconnect FAQ](https://github.com/docker-easyconnect/docker-easyconnect/blob/master/doc/faq.md) | Image-specific operational limitations / 镜像相关运行限制 |
| [OrbStack documentation](https://docs.orbstack.dev/) | macOS container runtime / macOS 容器运行时 |
| [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/) | Alternative runtime and installation prerequisites / 可选运行时与安装条件 |
| [Docker port publishing](https://docs.docker.com/engine/network/port-publishing/) | Host bindings and published-port exposure / 宿主机绑定与端口暴露 |
| [Shadowrocket App Store listing](https://apps.apple.com/us/app/shadowrocket/id932747118) | Application source and supported platform details / 应用来源及平台信息 |
| [Mihomo SOCKS proxy](https://wiki.metacubex.one/en/config/proxies/socks/) | SOCKS upstream configuration / SOCKS 上游配置 |
| [Mihomo rules](https://wiki.metacubex.one/en/config/rules/) | Rule order, domain/IP matching and `no-resolve` / 规则顺序、域名/IP 匹配与 `no-resolve` |
| [Mihomo general configuration](https://wiki.metacubex.one/en/config/general/) | Local listener and LAN exposure settings / 本机监听与 LAN 暴露配置 |
| [VS Code network connections](https://code.visualstudio.com/docs/setup/network) | Proxy, certificates and extension-specific behavior / 代理、证书与扩展差异 |
| [curl manual](https://curl.se/docs/manpage.html) | Explicit proxies, remote hostname resolution and bounded checks / 显式代理、远端解析及限时检查 |
| [Git configuration](https://git-scm.com/docs/git-config) | HTTP proxy scope and credential handling / HTTP 代理范围与凭据处理 |
| [OpenSSH client configuration](https://man.openbsd.org/ssh_config) | Host-specific ProxyCommand and host identity checks / 按 Host 代理与主机身份校验 |
| [GitHub Copilot network troubleshooting](https://docs.github.com/en/copilot/how-tos/troubleshoot-copilot/troubleshoot-network-errors) | Public service network diagnosis; consult the current endpoint requirements / 公共服务网络排查与当前端点要求 |
| [Copilot network configuration](https://docs.github.com/en/copilot/how-tos/copilot-in-your-ide/set-up-copilot/configure-network-settings) | Editor HTTP proxy settings and certificate handling / 编辑器 HTTP 代理及证书配置 |
| [Copilot allowlist](https://docs.github.com/en/copilot/reference/copilot-allowlist-reference) | Current service destinations; a case-specific domain is not the whole list / 当前服务目标，案例域名不等于完整清单 |
| [Copilot logs](https://docs.github.com/en/copilot/how-tos/troubleshoot-copilot/view-logs) | Supported diagnostic entry points / 受支持的诊断入口 |
| [Supported Copilot models](https://docs.github.com/en/copilot/reference/ai-models/supported-models) | Check present availability instead of treating historical model names as entitlement / 当前可用性，不以历史型号替代权限核对 |

Shadowrocket UI instructions describe observed application behavior, not a stable configuration API. Labels and available controls vary by version. The tutorial deliberately avoids direct edits to the application's private database.

Shadowrocket 操作步骤基于实际界面行为，不作为稳定配置 API 的承诺；控件与名称会随版本变化。本教程不采用直接修改其私有数据库的方法。

The original maintenance environment verified selected internal AI, SSH/Git and public model workflows. Published addresses and configuration are newly invented/sanitized examples. Configuration parsing and documentation checks do not constitute a fresh VPN deployment, an enterprise compatibility guarantee, UDP coverage, or unattended MFA recovery. Each user's actual application acceptance remains necessary.

源维护环境验收过特定内网 AI、SSH/Git 和公共模型流程。公开地址与配置是重新编写的虚构/脱敏示例。配置解析和文档检查不等于全新 VPN 部署验收，也不承诺所有企业兼容、UDP 覆盖或无人值守 MFA 恢复；每台机器仍须完成自己的实际应用验收。

This repository is independent documentation. It does not redistribute the aTrust client or other third-party binaries. Their licenses, service terms and organization access policies remain applicable.

本仓库为独立教程，不分发 aTrust 或其他第三方二进制，其许可证、服务条款及组织访问政策继续适用。

## Public template checks / 公开模板检查

Executed locally on 2026-10-01, before first publication / 于 2026-10-01 首次发布前在本地执行：

| Check / 检查 | Result / 结果 |
| --- | --- |
| Docker Compose `config -q`, v5.1.2, synthetic private environment file | PASS; no container started / 通过，未启动容器 |
| Compose with missing VNC password / 缺少 VNC 密码 | Correctly rejected / 按预期拒绝 |
| Mihomo `-t`, Meta 1.19.30, darwin arm64, isolated temporary directory | PASS; no live configuration changed / 通过，未改运行中配置 |
| JSON/YAML parsing and loopback/TUN/policy assertions / 解析与绑定、TUN、策略检查 | PASS |
| Local Markdown links/anchors, code-fence closure and shell syntax / 本地链接、锚点、代码块与 shell 语法 | PASS |
| Publication content review and private-data pattern/IP allowlist scan / 发布内容审查与敏感模式、IP 白名单扫描 | PASS |

These results apply to the initial templates. Re-run the relevant checks after changes. No connection to the fictional targets was attempted.

以上结果对应初始模板，后续修改需重新执行相关检查；没有尝试连接虚构目标。

The 2026-10-03 documentation update rechecks Markdown links/anchors, shell and JSON snippets, bilingual evidence consistency, and private-data patterns. Network configuration templates are unchanged. See the [Copilot case](copilot-claude.en.md) / [中文案例](copilot-claude.zh-CN.md) for the separately dated historical application results.

2026-10-03 文档更新重新检查 Markdown 链接/锚点、shell 与 JSON 片段、双语证据一致性和敏感信息模式；网络配置模板未变。历史应用验收另见上述专项案例中的明确日期。
