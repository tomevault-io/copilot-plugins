## exec-secrets

> Running child processes or handling credentials / secrets.


All child processes go through `src/core/services/exec.ts` - `run` streams output, `capture` collects stdout - both with `shell: false` and an explicit argv array. Never build a shell string or call `spawn` / `exec` directly.

- Secrets (`.p8` / `.p12` / keystore / private keys) live in the OS keychain via the secret store; `~/.launch` holds non-secret paths and ids only. Don't log, write, or commit key material.

---
> Source: [YosefHayim/launch-store](https://github.com/YosefHayim/launch-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
