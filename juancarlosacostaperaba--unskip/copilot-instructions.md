## unskip

> These instructions apply to AI-assisted changes throughout this repository.

# AI development rules

These instructions apply to AI-assisted changes throughout this repository.

## Scope and workflow

- Treat GitHub issues and their acceptance criteria as the source of truth.
- Work on one issue in a focused branch and pull request.
- Inspect existing decisions before changing code, keep diffs focused, and do not close incomplete issues.
- Record commands actually run and never claim an unexecuted check passed.
- Use English for code, tests, commits, and documentation.

## Architecture and quality

- Keep domain models and application rules in `Unskip.Core`.
- Keep WPF presentation in `Unskip.App`; do not put business logic in code-behind.
- Preserve test seams around process execution, filesystem paths, clocks, and persistence.
- Keep nullable references, analyzers, deterministic builds, and warnings-as-errors enabled.
- Prefer the BCL and maintained Microsoft/.NET packages; avoid speculative abstractions.

## Product, security, and privacy invariants

- Windows only; local-only user data; no server, cloud backend, telemetry, sound, Node.js, Electron, or embedded browser.
- Never invoke `cmd.exe`, PowerShell, or shell expansion from the application.
- Future `msg.exe` execution must use `ProcessStartInfo` with `UseShellExecute = false` and separate `ArgumentList` entries.
- Never execute message content and never commit real workplace names, hosts, IP addresses, credentials, or personal data.
- Never claim a message was read, displayed, or acknowledged. Use honest result statuses.
- Keep examples fictitious and state that Unskip is unofficial and unaffiliated with Microsoft or any employer.

<!-- codebase-memory-mcp:start -->
## Codebase Knowledge Graph

Prefer codebase-memory-mcp for code discovery in this order:

1. `search_graph`
2. `trace_path`
3. `get_code_snippet`
4. `query_graph`
5. `get_architecture`

Run `index_repository` when the project is not indexed. Fall back to text search for literals, errors, and non-code files.
<!-- codebase-memory-mcp:end -->

---
> Source: [JuanCarlosAcostaPeraba/unskip](https://github.com/JuanCarlosAcostaPeraba/unskip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
