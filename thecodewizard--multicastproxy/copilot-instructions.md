## multicastproxy

> Build and maintain a Windows service that relays multicast traffic between two Layer 3 networks by:

# AGENTS.md

## Project purpose
Build and maintain a Windows service that relays multicast traffic between two Layer 3 networks by:
1. receiving multicast locally,
2. tunneling payloads as unicast to a remote peer,
3. re-emitting them as multicast on the remote network,
4. optionally rewriting subnet references inside the payload.

This project must prioritize reliability, safe operations, simple configuration for non-technical users, and maintainable production code.

---

## Primary implementation goals
- Implement this as a Windows Service, not as a console-only utility.
- Prefer .NET Worker Service hosting with Windows Service integration.
- The service must be able to run unattended and recover cleanly from transient network failures.
- The same codebase should support both relay directions through configuration, not separate binaries unless clearly justified.

---

## Non-negotiable functional requirements
- Multicast listener must support one or more configured multicast source ports.
- Valid port ranges are 0-65535; reject invalid configuration with clear logs.
- A unicast tunnel port must be configurable for agent-to-agent forwarding.
- A destination IP must be configurable for the remote agent.
- Payload rewrite settings are optional:
  - If no rewrite source and destination subnet are configured, payloads must be forwarded unchanged.
  - If rewrite is enabled, rewrite only the intended subnet text and preserve all other payload bytes where possible.
- Do not assume a single multicast use case or hardcode customer-specific addresses in production code.

---

## Configuration rules
- Configuration must be simple for non-technical operators.
- Prefer `appsettings.json` plus environment-specific overrides.
- All important settings must be documented in a sample config file.
- Never hardcode production IPs, ports, secrets, or environment-specific paths.
- Validate configuration at startup and fail fast with human-readable Event Log messages.
- Use explicit typed options classes for configuration.

Suggested configuration model:
- `MulticastPorts`: array of integers
- `TunnelPort`: integer
- `DestinationIP`: string
- `MulticastGroup`: string if needed
- `ListenInterfaceIP`: optional string
- `SendInterfaceIP`: optional string
- `PayloadRewriteSourceSubnet`: optional string
- `PayloadRewriteDestinationSubnet`: optional string
- `Logging:LogLevel`

---

## Security and public-repo rules
- This repository is public. Never introduce secrets, API keys, tokens, credentials, private endpoints, or customer-specific addressing into source control.
- Assume all config files except example templates may contain sensitive environment details; avoid logging sensitive config values unless explicitly running in debug mode.
- Do not add any network-driven code execution, remote script download, or unsafe binary replacement logic.
- Keep all public examples on RFC documentation ranges and documentation-only values.

---

## Deployment and upgrade rules
- This service must not self-update.
- New versions are distributed as MSI packages and installed manually or through external deployment tooling.
- Do not implement GitHub polling, release checking, updater helpers, or in-place binary replacement unless explicitly requested later.
- Keep the runtime service focused on packet relay, configuration validation, and logging.

---

## Networking and protocol rules
- Use UDP sockets carefully and explicitly document socket options.
- Be conservative with multicast behavior; do not broaden multicast scope unless required.
- Avoid packet amplification, loops, or duplicate forwarding.
- Protect against relay loops between two agents. Add a loop-prevention strategy if traffic can return to the sender.
- Packet handling should be resilient: malformed packets should be logged at debug/warning level and should not crash the service.
- Preserve raw bytes unless a rewrite is explicitly configured and safely applicable.
- Do not assume payloads are always ASCII unless the protocol guarantees it. If text rewrite is attempted, handle non-text payloads safely.

---

## Windows service and operations rules
- Service must integrate cleanly with Windows Service Control Manager.
- Log operational events to Windows Event Log.
- Logging must be configurable so normal operation is quiet, while debug mode provides detailed packet-flow troubleshooting.
- Expected logs:
  - startup/shutdown
  - config validation failures
  - listener bind/join failures
  - remote send failures
  - packet counts / packet summaries at debug level
- Do not spam Event Log with per-packet informational logs in normal mode.

---

## Code quality rules
- Prioritize production-ready code over proof-of-concept shortcuts.
- Break the code into small services/classes with clear responsibilities.
- Avoid giant Program.cs implementations.
- Prefer the following separation:
  - configuration/options
  - multicast receive service
  - unicast tunnel service
  - multicast re-emission service
  - payload rewrite service
  - Windows service host/bootstrap
  - logging/event log integration
- Keep customer-specific constants out of business logic.
- Use dependency injection and typed options.
- Make cancellation and shutdown behavior explicit and correct.

---

## Testing and verification
Before considering work complete, always:
1. build the solution,
2. run tests,
3. verify startup configuration validation,
4. verify at least one no-rewrite forwarding case,
5. verify one rewrite-enabled case,
6. verify the service handles socket/network failure without crashing.

At minimum, add tests for:
- configuration validation
- subnet rewrite logic
- invalid rewrite configuration
- loop-prevention logic if implemented
- packet handling behavior that can be tested without live multicast infrastructure

If integration tests are added, keep them separate from fast unit tests.

---

## Change discipline
- Do not rewrite the whole project when a targeted change is sufficient.
- Preserve working behavior unless there is a clear bug or architecture reason to change it.
- When changing protocol handling, document the exact before/after behavior.
- When fixing a bug, add or update a test where practical.
- If you discover recurring guidance that should persist, update this `AGENTS.md`.
- Always run all the tests after every change.

---

## Output expectations for Codex
When implementing features:
- make reasonable assumptions and proceed,
- explain the assumptions clearly,
- report any protocol ambiguity,
- summarize risks,
- list what was verified.

When proposing architecture:
- prefer the simplest production-safe design,
- avoid unnecessary external dependencies,
- justify any new dependency.

When editing code:
- keep diffs focused,
- do not add placeholder TODO logic where working code is feasible,
- do not leave dead proof-of-concept code behind.

---

## Suggested initial architecture
Unless the repository already defines another pattern, prefer:
- .NET 8 Worker Service
- Generic Host with `UseWindowsService()`
- `appsettings.json` configuration
- typed options with validation
- `Microsoft.Extensions.Logging`
- Windows Event Log provider
- background services for receive/forward tasks
- unit tests for rewrite and config validation

---

## Acceptance criteria
A change is only complete when:
- it builds,
- tests pass,
- service behavior is documented,
- config is documented for non-technical operators,
- no secrets or environment-specific values are committed,
- logging is useful but not noisy in normal mode.

---
> Source: [thecodewizard/MulticastProxy](https://github.com/thecodewizard/MulticastProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
