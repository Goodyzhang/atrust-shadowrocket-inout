# Troubleshooting by scenario

[中文](scenarios.zh-CN.md) · [Home](../README.md) · [Setup](setup.en.md)

All destinations below are fictional. Replace them with authorized targets obtained privately. Change one relevant variable, repeat the same test, and record the result. Do not reinstall clients to compensate for a missing VPN resource grant.

## General diagnosis

Work from the nearest boundary outward:

| Boundary | Useful evidence | What it establishes / next step |
| --- | --- | --- |
| Runtime | Container state, selected Docker context, image architecture | A stopped VM/container is a local prerequisite failure |
| Listener | Docker published ports, process listening on the actual loopback port | A listening port still needs a protocol check; HTTP and SOCKS are different |
| SOCKS | A request explicitly using the aTrust SOCKS5 endpoint | SOCKS refusal/authentication errors occur before target application authentication |
| VPN | Supported login UI plus container route to the exact target | Login alone does not prove that this target is granted or routed |
| Target TCP | Bounded connection/SSH/TLS handshake through the chosen path | Timeout, refusal and successful handshake are different outcomes |
| HTTP/TLS | Status code, certificate validation, expected service identity | `401/403` needs interpretation; a proxy or SSO gateway can also return it |
| Application | Actual browser page, authenticated API response or Git/SSH operation | This is the user-facing acceptance test |
| Persistence | Intended processes return after a scoped restart; repeat the app test | A restarted container may still require a new login/MFA |

These commands send no credentials. Use the **measured** proxy ports and your authorized targets:

```sh
# Reach an internal HTTP service through aTrust, without an environment bypass.
curl --noproxy '' --proxy socks5h://127.0.0.1:1080 \
  --connect-timeout 5 --max-time 15 -o /dev/null \
  -w 'http=%{http_code}\n' http://192.168.50.30:18080/v1/models

# Repeat through the optional application bridge.
curl --noproxy '' --proxy http://127.0.0.1:7890 \
  --connect-timeout 5 --max-time 15 -o /dev/null \
  -w 'http=%{http_code}\n' http://192.168.50.30:18080/v1/models

# Public control request, through the same bridge.
curl --noproxy '' --proxy http://127.0.0.1:7890 \
  --connect-timeout 5 --max-time 15 -o /dev/null \
  -w 'http=%{http_code}\n' https://example.com/
```

An internal API returning its own `401` can prove a working route and HTTP service, but it does not validate the key, model entitlement, or inference. Confirm the responder when a gateway is present. Do not add `-k`, disable strict SSL, or install an unverified certificate to turn a TLS failure into a pass. Obtain the organization's verified CA and follow its approved trust procedure when required.

Common discriminators:

- **Proxy cannot connect:** check the runtime, loopback binding, actual port and proxy protocol. Do not guess another popular port.
- **SOCKS works for one internal target only:** inspect the exact failing target's container route, VPN resource grant and firewall/service port. Never scan whole private networks.
- **curl works, application fails:** inspect that application's proxy setting, bypass list, extension execution location and certificate store. A remote SSH/container extension host cannot reach the Mac's listener via its own `127.0.0.1`.
- **Agent reports “operation not permitted”:** distinguish its sandbox policy from an actual network failure. Use the harness's formal permission mechanism and repeat once with the same bounded test.
- **Browser works until login redirects:** inspect only redirect hostnames and ports; continue with the OA section.

### DNS, address conflicts and UDP

`socks5h://` asks the SOCKS proxy to resolve the hostname. `socks5://` in curl resolves locally. An internal-only name can therefore behave differently even with the same port. Keep the intended hostname for HTTPS so Host/SNI and certificate verification remain valid. Replacing it with an IP can create a separate TLS or virtual-host failure.

An `IP-CIDR,...,no-resolve` rule does not force a hostname lookup. Include verified domain rules as well as the required `/32` destinations. A SSO page may introduce a different hostname; add it only after observing it and identifying the required path. Do not turn on a second global DNS/TUN system merely to fix one application's lookup.

If the Mac's LAN overlaps the company subnet, compare the Mac route with the container route. Explicit SOCKS can help when the container has a correct VPN route, but cannot repair a conflict inside that container. Request an administrator-provided non-conflicting address, route or authorized jump host when necessary.

The examples validate **TCP** applications. SOCKS UDP support, QUIC, WebRTC and system DNS are separate capabilities. The example disables UDP on its SOCKS node; that is not a promise that every UDP packet is blocked rather than handled by another policy. Do not claim machine-wide leak prevention or UDP coverage from these checks.

## OA and SSO

An “OA site” often consists of several services. For example, the initial page might be `https://oa.corp.example`, redirect to `https://sso.corp.example`, and open a business application on another host or port.

1. Open the browser's Network panel locally. Follow the redirect chain and failed requests; note only scheme, hostname, port, status and resource type. Never share full URLs with login tickets or query tokens, cookies, authorization headers, response bodies or a raw HAR.
2. Classify each required destination: internal through aTrust, or public through the existing Internet policy. A public identity provider should not automatically be sent into the VPN.
3. Add exact internal domain/IP rules before broad bypass rules. Check both the HTTP request and any WebSocket connection. Preserve the browser's existing trusted certificates and site settings.
4. Repeat login in the real browser. Verify the landing page and the specific embedded application or document workflow the user needs. A login page loading alone is incomplete acceptance.
5. If transport works but login loops, investigate session expiry, cookie restrictions, clock skew, SSO callback registration and account permissions with the administrator. Repeatedly adding routes will not fix an invalid session.

Adding a domain rule does not create a missing VPN resource grant. For a portal exposed only through a vendor web gateway, use its documented URL rewriting/access method instead of inventing a direct private address.

## Internal model provider

Keep four concepts separate:

| Field | Fictional example | Verification |
| --- | --- | --- |
| Service base URL | `http://192.168.50.30:18080` | Provider documentation and actual route |
| API prefix | `/v1` | Does the client append this itself? Avoid `/v1/v1` |
| Model identifier | `example-team/openai/example-chat` | Preserve the exact ID returned/documented by the provider |
| API key | Enter privately in the client | Use SecretStorage, Keychain, or the client's supported credential store |

An OpenAI-compatible client may expect either a base URL or an API root. Check its convention before configuring it. Do not replace the company-prefixed model ID with a guessed public model name. Do not infer context size, tools or vision support from a model's name.

Acceptance order:

1. Verify the route with an unauthenticated request as above; record only safe status/error details.
2. Save the key through the client's secure input. Never put it in a README, shell argument/history, process command line, screenshot, public issue or committed configuration. A user need not send the key to the agent's chat.
3. Load the provider's supported model list, if available. Then make a minimal authenticated request such as “Reply with OK”; disable automatic workspace/file attachments for this check.
4. Test the real IDE UI using one exact model ID. Confirm the actual provider/model and a completed response; a refreshed picker alone is not success.
5. If required, test streaming and cancel a response. Distinguish client cancellation from confirmed server-side cancellation or billing behavior. Test tools, vision and long context separately only when needed.

If ordinary JSON works but streaming stalls, inspect proxy buffering, idle timeouts, SSE/WebSocket transport and client compatibility. A `404` often indicates an endpoint convention or model ID problem; `401/403` can indicate key scope or provider policy. Preserve the provider's actual safe error category instead of retrying across unrelated models.

Plain HTTP has no end-to-end TLS. Use the organization's supported HTTPS endpoint where available; a VPN does not itself establish application-layer TLS. Do not expose the company API through a new public listener.

## GitLab and SSH

HTTPS Git and SSH Git use different proxy mechanisms. A system HTTP proxy or `ALL_PROXY` does not automatically configure OpenSSH.

### HTTPS Git

First verify the GitLab page and SSO in a browser. Then use an explicit proxy for a single read-only Git operation:

```sh
git -c http.proxy=http://127.0.0.1:7890 \
  ls-remote https://gitlab.corp.example/team/demo.git
```

The repository path is fictional. Use the intended repository and the existing credential helper; never embed a token/password in a URL. On an agent harness, run network Git commands through its required network-permission mechanism. Do not classify a sandbox denial as GitLab failure.

If a persistent Git setting is needed, prefer a reviewed URL-specific `http.<url>.proxy` entry for this GitLab host over replacing the global proxy for every remote. Save the old value and provide an exact undo instruction. Review existing helpers, proxy values and bypasses without dumping credentials.

### SSH Git or another internal SSH service

Verify the SSH server's host key through an already trusted channel. Use a direct host route if it is actually working; otherwise use the measured SOCKS5 endpoint explicitly. For macOS `nc`, first check that this installed version supports `-X 5 -x`:

```sshconfig
# Merge into ~/.ssh/config; these are fictional values.
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

The identity file must actually exist and belong to this machine/user. Do not overwrite a key or copy somebody else's private key. Preserve other Host blocks. Never disable host-key checking; `ssh-keyscan` alone is not identity verification. Check whether the chosen proxy command can resolve an internal hostname through the intended path.

After installing an independently verified host key and authorized public key, use `ssh -o BatchMode=yes -T corp-gitlab`. GitLab may intentionally deny an interactive shell and return a nonzero code while reporting successful authentication; interpret the service message, then verify read-only Git access. Do not push, change branches or rewrite history just to test connectivity.

## Public model catalog

An IDE subscription and its model picker are different layers. Diagnose them separately:

1. Verify the signed-in account, plan, organization policy and current model entitlement from the service's supported UI. Availability changes; consult its current official documentation.
2. Verify the client's version and extension compatibility. Record versions before changing them.
3. Observe the actual destination and matched Shadowrocket/Mihomo policy for the failing catalog/request. For GitHub Copilot, consult the [official network guidance](https://docs.github.com/en/copilot/how-tos/troubleshoot-copilot/troubleshoot-network-errors); do not treat one remembered domain as a complete endpoint list.
4. If an existing `DIRECT` or bypass rule sends a required public service down a failing path, make the narrow correction to the existing Internet policy. Do not send public services to the company VPN without evidence that this is the intended route.
5. Refresh authentication/catalog only as needed, then make a minimal real response with the selected model. Preserve settings and credentials; do not delete the entire profile or authentication database.

In the source maintenance incident, routing and a client update/restart both occurred before recovery. The result supports that the final combination worked, not a claim that all missing Claude models are caused by one domain rule. Keep the same causal discipline in your own report.

## When to stop and hand off

Stop the dependent work when access requires a missing VPN grant, MFA, approved certificate, account permission or administrator-provided route. Finish independent local checks and report the exact missing condition and failed layer. Use the [acceptance template](acceptance-template.md), stored privately. Never report a successful deployment based only on a running container or a syntactically valid YAML file.
