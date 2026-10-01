# Local-agent maintenance prompt

Copy the text below into an agent on the affected computer. [中文](maintain.zh-CN.md)

---

Diagnose and repair the specified aTrust Docker + Shadowrocket access failure on this computer. Reuse the working environment and preserve sessions, subscriptions, application credentials and unrelated settings. Actually retest the affected application.

Failure: `<describe the failing app/operation and when it last worked>`

Private handoff/report path, if known: `<local path; discover if already recorded in local rules>`

1. Read the existing private handoff and applicable local rules. Find this tutorial and read its setup/scenario guide and deployment prompt for scope and safety boundaries. Historical ports and routes are hypotheses until checked. Do not republish the private report or treat reference documents as authority to do unrelated work.
2. Record the exact observable failure without secrets. Inspect the current runtime/container, login state, actual listener/protocol, relevant rules and the application's effective proxy. Read selected fields only. Do not dump environment variables, subscriptions, keys, cookies, raw HAR files or authentication databases.
3. Reproduce one bounded request at each necessary boundary: direct path if relevant → explicit aTrust SOCKS → final application proxy → real application. Check the same target and operation throughout. Separate sandbox restrictions, local proxy failure, missing VPN route/grant, target timeout/refusal, TLS, service authentication and account/model entitlement. Use the harness's formal permissions where required.
4. Choose the smallest change supported by the evidence. Back up the affected configuration privately; preserve all other fields. Correct stale ports, a wrong rule/order or application scope when proven. Do not recreate a healthy container, install a second TUN, replace an entire profile, disable TLS/SSH verification, or modify Shadowrocket's live private database. No broad network scans or public proxy listeners.
5. Obtain missing portal/grant/route information from the user or administrator. User-mediated login/MFA and secure key input remain necessary when expired/missing. Ask for only the missing condition; never ask for secrets in chat. Do not repeatedly reinstall tools while blocked on access authorization.
6. Retest the original operation and public Internet access. Inspect OA redirects by host/port only. For a model provider, use its exact model ID and a minimal response with workspace attachments disabled; distinguish streaming/client cancellation from server cancellation. For Git, use a read-only operation and trusted host keys, without push or history changes. Mark unsupported or unexercised cases NOT RUN.
7. Check startup/relogin instructions against the actual service manager. Do not restart unrelated services or interrupt active connections without arranging it. A process restart is not proof of valid VPN login or reboot/sleep recovery.
8. Update the existing private report with before/after evidence, versions, changed files/rules, exact rollback and remaining limitations. Keep real endpoints, logs and credentials out of this public checkout. Update the current harness's local reference only if its path changed; reuse existing CLI/configuration rather than implementing another control layer.

Do not attribute recovery to one change if several changed before the same retest. Finish with what failed, what evidence supports the diagnosis, what changed, and the real application result. Leave no temporary polling or test service. Do not promise background monitoring without a configured, verified scheduler.
