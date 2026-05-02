## security

> Security rules (non-negotiable)


# Security rules (non-negotiable)

- Credentials live in the OS keychain (`src/keychain/`) or the
  encrypted fallback. **Never** in a config file at rest.
- All log output routed through `src/security/redact.ts`. Do not
  `console.log` a config object that may contain an API key.
- Prompt-injection defence uses fenced-data boundaries
  (`src/security/injection.ts`). Any new place that ingests untrusted
  content must be fenced.
- Never add a code path that writes outside the sandbox without a
  matching permission gate. Never bypass permission prompts unless
  the user explicitly passed `--skip-permissions` or `--allow-*`.
- Shell risk is classified in `src/sandbox/shell.ts`. The `critical`
  tier is hard-blocked — do not add exceptions.

---
> Source: [hoangsonww/Forge-Agentic-Coding-CLI](https://github.com/hoangsonww/Forge-Agentic-Coding-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
