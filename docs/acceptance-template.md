# Private acceptance report / 私有验收报告

**Copy outside this checkout before filling in real information. Do not commit the completed report.**

**填写真实信息前复制到仓库外，不提交填好的报告。**

- Date / 日期、时区：
- OS, architecture, relevant app versions / 系统、架构、相关应用版本：
- Requested applications and operations / 指定应用与操作：
- Existing environment reused / 复用的已有组件：
- Actual selected path, including protocols / 实际链路与协议：
- Loopback listeners and owning processes / 本地监听与进程：
- Active config paths / 生效配置路径：
- Changed settings/rules / 修改的设置与规则：
- Private backups and exact rollback / 私有备份位置及精确回滚：
- Startup and VPN relogin sequence / 启动与 VPN 重新登录顺序：
- Harness rule reference, if actually configured / 已实际配置的 harness 本地规则引用：

Use PASS / FAIL / NOT RUN. Record timestamp, test execution location, safe command or UI action, observed result, and its limit. A simulated or syntax-only check is not a network pass. Never include passwords, API keys, cookies, authorization headers, login tickets, private keys or full raw logs.

使用 PASS / FAIL / NOT RUN。记录时间、测试执行位置、不含秘密的命令或 UI 操作、观察结果与边界。模拟和语法检查不能算网络通过。不填写密码、API Key、Cookie、授权头、登录票据、私钥或完整原始日志。

| Check / 项目 | Status / 状态 | Evidence and limits / 证据与边界 |
| --- | --- | --- |
| Runtime, image architecture / 运行时、镜像架构 | NOT RUN | |
| Loopback binding and protocol / 本机绑定与协议 | NOT RUN | |
| VPN login / VPN 登录 | NOT RUN | |
| Explicit SOCKS to required target / 显式 SOCKS 到指定目标 | NOT RUN | |
| Final proxy route and matched policy / 最终路径与命中策略 | NOT RUN | |
| Target TLS/service identity / TLS 与服务身份 | NOT RUN | |
| Public Internet control / 外网对照 | NOT RUN | |
| OA landing page and embedded app / OA 落地页与嵌入应用 | NOT RUN | |
| Exact provider/model authenticated response / 指定模型已认证响应 | NOT RUN | |
| Streaming and client cancellation / 流式与客户端取消 | NOT RUN | |
| Git read-only access / Git 只读访问 | NOT RUN | |
| SSH host identity and authentication / SSH 主机身份与认证 | NOT RUN | |
| Scoped service restart / 有限范围服务重启 | NOT RUN | |
| Reboot/sleep recovery / 系统重启、休眠恢复 | NOT RUN | |
| Other requested capabilities / 其他指定能力 | NOT RUN | |

## Remaining conditions / 剩余条件

- Concrete blocker, owner and required next input / 具体阻塞、负责方和下一项输入：
- Untested capabilities and why / 未测能力及原因：
- Concurrent changes that prevent a single-cause conclusion / 无法单项归因的并发变更：
- Temporary test resources cleaned up / 临时测试资源清理情况：
- Monitoring / 持续监控：not configured unless separately verified / 未单独验收则视为未配置。
