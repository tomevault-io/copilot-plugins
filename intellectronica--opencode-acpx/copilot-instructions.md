## opencode-acpx

> - Use British English in documentation and user-visible strings.

# Repository instructions

- Use British English in documentation and user-visible strings.
- Use pnpm for dependency management and scripts.
- Keep the plugin compatible with OpenCode 1.18.15 or newer within major version 1.
- Pin Acpx and protocol dependencies exactly; upgrades require contract and real-agent tests.
- Keep ACP processes and credentials isolated in the plugin-owned worker.
- Fail closed for permissions, elicitation, malformed messages, and unsupported capabilities.
- Never retry a turn automatically after observable side effects.
- Add deterministic tests for every protocol or lifecycle behaviour.
- Use Conventional Commit subjects and keep commits focused.

---
> Source: [intellectronica/opencode-acpx](https://github.com/intellectronica/opencode-acpx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
