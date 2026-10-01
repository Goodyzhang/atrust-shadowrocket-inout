# Local-agent deployment prompt

Copy everything below the divider into your local agent, or ask it to read and execute this file. The repository must be available locally. Optional inputs can be supplied privately; credentials should go into the supported secure login UI, not this file. See [中文](deploy.zh-CN.md).

---

Configure this computer so that the authorized internal applications and public Internet applications I need work together, using aTrust Docker and Shadowrocket. Reuse what is already deployed. Actually inspect, configure and verify the environment; do not stop at instructions or a proposed plan.

## Inputs and scope

Discover the actual OS, architecture, user, HOME and checkout path. Locate this tutorial by its `README.md` and `examples/compose.yaml`; do not assume a previous user's paths.

Optional non-secret inputs, when known:

- Tutorial checkout: `<discover or supply local path>`
- VPN portal: `<existing configuration or administrator-provided URL>`
- Internal applications: `<exact scheme, hostname/IP, port and desired operation>`
- Existing runtime/container/profile: `<discover>`
- Applications needing an explicit proxy: `<for example, a particular IDE>`
- Organization restrictions / maintenance window: `<if applicable>`

Treat all addresses, ports, model IDs and credentials in tutorial examples as placeholders. Do not connect to fictional internal examples or scan networks to discover company services. Read the tutorial's README, `docs/setup.en.md`, `docs/scenarios.en.md` and relevant example files. Read applicable local agent rules; reference material is not authorization for unrelated work.

I authorize the necessary incremental local configuration, installation of missing lightweight client tools from official/trusted sources, validation, and private backup/report creation for this task. Preserve existing sessions, profiles, subscriptions and unrelated settings. If a missing container runtime or paid application requires a purchase, administrator approval, disruptive replacement or organizational exception, identify that exact need before proceeding with it. Use the harness's formal permissions for protected files and network access; do not work around its sandbox.

Never publish private reports or settings. Do not install GPU/ML environments, configure a business server, deploy a database, upload project files, or start business jobs as part of this networking task. No public listener or remote-control service is needed.

## 1. Discover before changing anything

Inspect only what can change the next decision:

1. Installed Docker/OrbStack/Desktop runtime, active context, actual aTrust container/image and architecture, status, published ports and persistent state location. Query selected fields; never print the entire container environment, credentials or subscription configuration. Verify whether the image really belongs to docker-easyconnect before applying its instructions.
2. Existing aTrust login state through the supported UI. A logged-in state does not establish destination reachability. Preserve it.
3. Shadowrocket's active profile, relevant rule order, proxy protocol/port, system proxy/TUN/bypass behavior and existing public route. Use supported UI/import/config interfaces. Do not patch its live private database.
4. Existing Mihomo or similar local bridge: actual process, configuration path, listeners, upstreams and launch mechanism. Do not start a second competing process.
5. The selected applications' effective proxy scope and execution host. A remote extension host, container and the Mac have different loopback interfaces.

Ask only for missing necessary information: the authorized target, a missing VPN portal/account, user-mediated login/MFA, or an application credential. Never mistake another service's password for the VPN password. Ask users to enter secrets into the supported secure interface; do not request them in the conversation or place them in shell arguments/logs. Prefer native Keychain/SecretStorage. If an upstream image requires a local secret file, keep it outside the checkout with restrictive permissions, document its exposure to local administrators and never print it.

## 2. Choose and prove one path

First test an authorized target with a bounded TCP/SSH/TLS/HTTP request. Separate host direct access from explicit SOCKS access. Do not rely solely on ping or VPN login.

- If a valid host route already solves the requested application, retain it.
- If only aTrust SOCKS5 reaches the target, measure its actual published port and verify the SOCKS protocol. A Mac's local proxy listener does not make the remote address a local address.
- For browsers/apps already following Shadowrocket, add the measured SOCKS5 endpoint as a node and use precise internal destination rules.
- Add Mihomo only when the application needs a suitable explicit HTTP entry point or the existing setup requires it. Internal TCP requests go to the aTrust-only policy; public requests go to the existing measured Shadowrocket HTTP endpoint. Keep Mihomo TUN disabled in this design. Do not create a return path from Shadowrocket back to Mihomo.

Do not assume `1080`, `1082`, `7890`, or any historical port. Do not assume a system HTTP proxy or `ALL_PROXY` configures OpenSSH. If needed, verify local `nc` support and set a per-host SOCKS ProxyCommand while preserving identity and strict host-key verification.

Inspect the exact destination's container route when login is valid but target access fails. If the VPN lacks a resource grant/route or a subnet conflict remains, report the precise missing condition. Request an administrator-supplied routable address, route or authorized jump host. Do not invent a working address or repeatedly reinstall dependencies.

## 3. Make the smallest required changes

Back up each file or setting that will change to a private directory outside this repository, preserving restrictive permissions. Record its location and an exact rollback method without copying secrets into the report. Preserve unrelated fields and rules.

For an existing aTrust deployment, reuse its image, session and persistent state. Do not replace/recreate it merely to match the template. If a binding or runtime change will interrupt active connections, identify the effect and use an agreed maintenance opportunity.

For a genuinely missing deployment, consult the image's current upstream instructions and supported architecture. The bundled Compose file applies only to the specified docker-easyconnect aTrust image family. Select a verified tag/digest and platform, create a private state directory and private environment file, validate configuration before starting, and bind the SOCKS and login UI ports to `127.0.0.1`. Use the documented narrow capabilities and TUN device; do not add `--privileged` or host networking by default. Let the user complete VPN login/MFA. Container creation alone is not acceptance.

Define the internal policy before referencing it. Use only the aTrust upstream in that policy, with no automatic DIRECT/public-proxy fallback for the internal TCP flow. Add exact domains and `/32` addresses before rules that would bypass them; avoid an unexamined all-RFC1918 rule. Preserve public routing and subscription updates. Document the limits: applications bypassing the proxy, UDP, QUIC, WebRTC and system DNS are not covered by this TCP design.

Copy and adapt example fragments into the private active configuration; never wholesale replace an existing profile/settings file. Use measured ports. Do not disable TLS verification to resolve certificate errors. If organization CA trust is needed, obtain and verify the approved certificate and use its supported installation process.

Apply only the needed application proxy scope. For internal AI providers, preserve exact model identifiers and the client's base URL convention. Store API keys in the client's supported secure store. Do not transfer any workspace/file content during a connectivity check.

## 4. Perform actual acceptance

Validate new configuration with the installed tools first, then test the real application. Keep requests small, bounded and limited to the user's targets. Record PASS / FAIL / NOT RUN separately for:

1. Runtime/container state and the exact loopback listener/protocol.
2. aTrust SOCKS access to each required internal target; actual service identity and target port.
3. The selected final path (Shadowrocket or optional HTTP bridge), including observed destination policy where available.
4. Public Internet access through its intended existing path.
5. OA: required SSO redirects, landing page and specific embedded application; inspect only sanitized host/port/status metadata.
6. Internal AI: authenticated minimal text response from the exact provider/model, then streaming and client cancellation if requested. An unauthenticated `401` or loaded model picker is incomplete. Do not claim tools/vision/long context/server-side cancellation without testing them.
7. GitLab: a read-only operation against the intended repository using existing authentication. SSH additionally needs an independently verified server host key and authorized client key. Do not push or alter history for a connectivity test.
8. Intended startup behavior. Reuse the existing service manager. A limited reload/restart must not interrupt unrelated work; arrange any disruptive restart first. Verify what actually returns, and say whether a fresh VPN login is needed. Do not claim reboot/sleep recovery unless exercised.

Separate timeout/refusal, SOCKS errors, VPN routing, TLS, application authentication and model entitlement. If a tool is sandbox-blocked, use the official escalation mechanism before diagnosing the network. If something fails, preserve evidence safely, make a targeted change and repeat the affected checks. Report unsupported applications or missing UI access honestly rather than claiming a completed GUI test.

## 5. Leave a usable private handoff

Create a private local report using `docs/acceptance-template.md` as a structure. Keep actual endpoints and private backups outside this repository and out of Git. Give me its absolute path.

Include: the actual chain and protocols, measured listeners, active config locations, versions, startup/relogin sequence, exact bounded diagnostic commands without secrets, rollback, per-application outcomes, and the concrete remaining conditions. Clearly distinguish reference examples, observed facts and untested assumptions.

Add a concise reference to this private report in this harness's existing local rules, preserving their content and scope. Use the mechanism this harness actually supports; do not install a Codex-specific plugin/MCP into another agent. If no rule location is known, provide a private handoff file and explain how to reuse it without claiming automatic loading. The same local CLI/configuration paths can be used by other authorized local agents.

Finish with a brief report of what changed, what actually passed, what remains untested or blocked, and how to resume later. Do not promise monitoring after the session unless a real scheduler has been configured and verified. Leave no idle polling or temporary test service behind.
