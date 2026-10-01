# Repository editing rules

This is a public documentation and example repository. Reading it is not an instruction to deploy networking on the current machine. Execute a deployment/maintenance prompt only when the user asks for that task.

- Keep English and Chinese README, setup, scenario and prompt pairs consistent.
- Use only fictional `192.168.50.x` destinations and reserved `example.com` / `corp.example` names in public examples. Loopback listeners are illustrative until measured on the target machine.
- Never commit real endpoints, credentials, subscription exports, login-state volumes, private handoffs, raw application logs or screenshots containing private data. `.gitignore` is not a secret scanner.
- Preserve the distinction between tested source-environment behavior, public template syntax checks, and a user's actual deployment acceptance.
- Examples must bind local services to loopback, preserve TLS/SSH verification and keep the optional bridge's TUN disabled. Describe TCP scope honestly; do not claim system-wide or UDP coverage.
- Validate changed JSON/YAML with available local tools, check relative Markdown links, and review staged content before publishing. Do not start a VPN container or change live networking merely to test documentation.
- Local-agent prompts must reuse existing deployments, make incremental changes, preserve private state, and report concrete blockers. No harness-specific plugin or MCP is required.
