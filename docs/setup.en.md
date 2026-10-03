# Setup and restart guide

[中文](setup.zh-CN.md) · [Overview](../README.md) · [Scenarios](scenarios.en.md)

All addresses below are examples. Perform discovery first and substitute your verified values. Run terminal commands on the Mac unless a command explicitly says `docker exec`. Do not run the example private addresses as a discovery scan.

## 1. Inventory the existing environment

Record your username, home directory, OS and CPU architecture. Check the runtime without changing its active context:

```sh
id -un
printf '%s\n' "$HOME"
uname -m
docker context show
docker ps --format '{{.Names}}\t{{.Image}}\t{{.Ports}}'
```

For the container you actually identified, inspect only the fields needed:

```sh
ATRUST_CONTAINER=your-existing-container
docker inspect "$ATRUST_CONTAINER" --format '{{.Config.Image}}'
docker port "$ATRUST_CONTAINER"
```

Use `docker image inspect <observed-image> --format '{{.Os}}/{{.Architecture}}'` to check the installed image. Compare that with `uname -m`. Upstream aTrust images support multiple architectures, so do not force amd64 on Apple Silicon without a reason. Use emulation only when the selected image requires it and the runtime supports it.

In Shadowrocket, record the active configuration, routing mode, selected public policy, local HTTP/SOCKS listeners and any LAN/TUN bypass rules. Read only relevant fields. Do not dump container environment variables, subscriptions, login URLs, private keys or entire app databases into an agent transcript.

Check actual listeners, for example:

```sh
lsof -nP -iTCP:1080 -sTCP:LISTEN
lsof -nP -iTCP:1082 -sTCP:LISTEN
lsof -nP -iTCP:7890 -sTCP:LISTEN
```

A listener proves a process accepted a bind, not that it speaks the expected proxy protocol or that the VPN is authenticated. Inspect the relevant system proxy bypass list as well: some applications skip proxies for private addresses before a routing rule can run.

Export/duplicate the active Shadowrocket profile through the app and save a private backup. Back up any application settings or Mihomo file you will edit. Keep backups outside Git; a mode-600 export can still contain subscriptions and credentials.

## 2. Reuse aTrust, or install it only when missing

### Existing installation

Keep the current container, image, mounted state, portal and login. Open its supported login UI if the session expired. Ask only for missing portal/account/MFA/certificate information. A server's SSH password is not a VPN password.

If the existing mapping binds `0.0.0.0` or `::`, record that it may expose a proxy or login UI to other devices. Our fresh-install example binds loopback only. Changing an existing Docker port mapping normally requires a container recreation: preserve mounts/login state, explain the interruption, and schedule that separately if access is currently in use. Do not recreate a working container merely to reproduce this tutorial.

### Fresh installation

Use one supported container runtime and check the [upstream aTrust instructions](https://github.com/docker-easyconnect/docker-easyconnect#readme). The [Compose file](../examples/compose.yaml) is for `hagb/docker-atrust`; other images require their own documentation.

1. Create private directories outside the repository:

   ```sh
   mkdir -p "$HOME/.config/atrust-inout" "$HOME/.local/share/atrust-inout/atrust"
   chmod 700 "$HOME/.config/atrust-inout" "$HOME/.local/share/atrust-inout/atrust"
   cp examples/atrust.env.example "$HOME/.config/atrust-inout/atrust.env"
   chmod 600 "$HOME/.config/atrust-inout/atrust.env"
   ```

2. Privately edit that external file. Set the supported platform, an existing absolute state directory, the chosen image, free ports and a local VNC password. The upstream VNC password option has an eight-character maximum; it is distinct from your VPN password. Do not put a VPN password into Compose or command arguments. Environment-backed VNC credentials are visible to Docker administrators; loopback binding does not protect against other users/processes on the same host.
3. Confirm the image architecture/tag with the registry/runtime. Pull the chosen image only if needed. Record the tested version or immutable image reference after validation; `latest` is just the template's starting point.
4. Check that loopback ports `1080`, `5901` and the aTrust web-login callback port `54631` are free. Preserve the callback mapping as documented upstream; another aTrust client can conflict with it.
5. Validate without printing resolved secrets, then start:

   ```sh
   docker compose -p atrust-inout --env-file "$HOME/.config/atrust-inout/atrust.env" -f examples/compose.yaml config -q
   docker compose -p atrust-inout --env-file "$HOME/.config/atrust-inout/atrust.env" -f examples/compose.yaml up -d
   docker compose -p atrust-inout --env-file "$HOME/.config/atrust-inout/atrust.env" -f examples/compose.yaml ps
   ```

6. Connect a VNC client to the configured loopback VNC port. Enter your administrator-provided VPN portal and credentials in the aTrust UI. Complete MFA yourself. If web login is required, follow the image's supported callback/browser procedure. A noVNC/browser image variant is a separate choice, not implied by this template.

The template grants `NET_ADMIN` and `/dev/net/tun` to the container, as required by this image; it does not request `--privileged` or host networking. Persistent `/root` data is private login state. Do not delete it to troubleshoot a connection problem. See [upstream usage](https://github.com/docker-easyconnect/docker-easyconnect/blob/master/doc/usage.md).

## 3. Prove the VPN path before editing application routing

Choose one administrator-approved target. For the fictional model service:

```sh
ATRUST_SOCKS_PORT=1080
curl --noproxy '' --proxy "socks5h://127.0.0.1:${ATRUST_SOCKS_PORT}" \
  --connect-timeout 5 --max-time 15 -sS -o /dev/null -w 'HTTP=%{http_code}\n' \
  http://192.168.50.30:18080/v1/models
```

The empty `--noproxy` prevents an inherited `NO_PROXY` value from invalidating this particular probe. `socks5h` delegates hostname resolution to the SOCKS proxy when the target is a hostname. This says nothing about how other clients resolve names. Curl's exit status must also be checked. `401` may establish transport plus an authentication boundary; `404` may identify a wrong path. Neither is a model acceptance test.

If it fails, compare the container's route with the Mac's route:

```sh
docker exec "$ATRUST_CONTAINER" ip route get 192.168.50.30
route -n get 192.168.50.30
```

Do not assume the interface is always named `utun7`. A route alone does not prove the destination accepts connections. A route visible only inside the container is sufficient for a SOCKS path but not for direct Mac access. If the VPN does not expose that resource, request a routable address/resource grant or authorized jump host. A local and remote subnet collision must be resolved deliberately; do not repeatedly reinstall clients.

## 4. Choose the smallest routing change

### A. Shadowrocket only

1. In the app, add a SOCKS5 node pointing to the **measured** aTrust loopback address and port. Use the local proxy's authentication if enabled; do not reuse VPN credentials as SOCKS credentials.
2. Name the node or a single-member policy `COMPANY-VPN`. Keep its only upstream as aTrust, with no direct/public fallback.
3. Merge [the example rule fragment](../examples/shadowrocket.rules.example) into a duplicate of your active profile after replacing the fictional destinations. Exact domains or `/32` addresses come before conflicting broad LAN, GEOIP, or final rules. Preserve existing public policies.
4. Select configuration-based routing and confirm the intended profile is active. Use the app's rule-test/connection view to check the destination's **effective** policy.
5. Test the actual browser page and a public site. If the browser bypasses private addresses before the proxy, adjust only the relevant bypass setting or choose a verified TUN/app path. Check that a TUN does not recapture its own upstream traffic.

UI labels and rule capabilities vary by release; do not edit the app's live SQLite database. A saved rule in an inactive profile changes nothing.

### B. Optional Mihomo application bridge

Use this when an application needs an explicit HTTP proxy or its internal request stack ignores the existing system path.

1. Reuse an existing Mihomo installation. If missing and Homebrew is available, inspect the formula before installing `mihomo`; otherwise use the [upstream installation guidance](https://wiki.metacubex.one/en/start/).
2. Save a **private** adapted copy of [the example](../examples/mihomo.example.yaml). It assumes a verified aTrust SOCKS5 endpoint and a verified Shadowrocket HTTP listener. Add proxy authentication privately if either requires it. Do not overwrite an existing configuration wholesale.
3. Validate the private file with `mihomo -t -f /absolute/private/path/config.yaml`. This tests configuration, not target reachability.
4. For a foreground first run, use `mihomo -f /absolute/private/path/config.yaml`. Stop only that test process after checking it. Do not start it over an already occupied port.
5. If using the Homebrew service, inspect `brew services info mihomo` and its actual configuration path, merge changes into that file, then start/restart only that service. Do not assume the service will use the private file passed to a previous foreground command. Check its live listener and logs afterward.
6. Merge [the VS Code settings fragment](../examples/vscode-settings.example.json) into the appropriate user settings and reload an idle window. Other apps need their supported proxy setting. Keep certificate verification on; install your organization's CA through its supported process if required.

No second global TUN is needed. The example has no controller API and adds no system-wide proxy setting. For public traffic it delegates to Shadowrocket; Shadowrocket must not send that traffic back to the Mihomo entrance. DNS and subnet conflicts still need their own diagnosis. See [Mihomo routing](https://wiki.metacubex.one/en/config/rules/) and [VS Code networking](https://code.visualstudio.com/docs/setup/network).

## 5. Acceptance, restart and rollback

For missing/unusable Claude models in VS Code Copilot, continue with the [dedicated repair and acceptance guide](copilot-claude.en.md). An effective HTTP proxy setting still needs catalog and real model-generation verification.

Use the [acceptance template](acceptance-template.md) outside the repository. Verify an internal application and a public application at the same time. Save only concise, redacted evidence; inspect real addresses privately.

Restart sequence: container runtime → aTrust container → valid VPN session → Shadowrocket profile/listener → optional Mihomo → client. `restart: unless-stopped` does not start the Mac runtime or satisfy MFA. After sleep or reboot, recheck the endpoints; do not advertise an autostart or recovery guarantee you have not tested.

Rollback one layer at a time: restore the touched app proxy fields, restore only the added/changed routing entries or select the saved profile, and stop only a newly introduced bridge if appropriate. Restore prior values rather than deleting unrelated user settings. If you created the sample stack, its matching `docker compose ... down` stops/removes that stack's containers; omit `-v` and preserve its external state directory. Never run this against a pre-existing user's stack.
