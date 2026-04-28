## teqo-legacy

> Never commit or push without explicit user request


# Commit/Push Permission Rule

Do not run any commit or push operation unless the user explicitly asks for it in this conversation.

## Required behavior

- Never run `git commit` proactively.
- Never run `git push` proactively.
- Never infer permission from context or previous sessions.
- If the user asks for code changes only, stop after edits and verification.
- If uncertain whether permission was granted, ask a clarifying question first.

## Allowed actions without commit/push permission

- Edit files
- Run tests, lint, build, and local verification
- Show `git status` / `git diff`
- Prepare a suggested commit message for user approval

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fsolla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
