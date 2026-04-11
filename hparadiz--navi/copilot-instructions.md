## navi

> - When debugging provider failures in this repo, check `logs/provider-calls.log` for the exact request payload, target, response body, and provider error message.

## Repo Notes

- When debugging provider failures in this repo, check `logs/provider-calls.log` for the exact request payload, target, response body, and provider error message.
- When debugging command execution or build/test failures in this repo, check `logs/shell-exec.log` for the executed command, policy outcome, exit code, stdout, and stderr.
- These logs are written automatically when Navi is launched from a workspace that contains a `Makefile`.
- If a user reports that a recent code or prompt change is not taking effect, verify whether an older daemon or TUI instance is still running before assuming the current binary is wrong.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hparadiz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hparadiz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
