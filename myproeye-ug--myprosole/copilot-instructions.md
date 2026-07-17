## security

> Security practices for secrets and user-uploaded files


# Security

Protect secrets and handle untrusted input safely.

- NEVER hardcode or commit secrets, API keys, or tokens. Keep them in gitignored files or environment variables and read them at runtime.
- Treat the existing `n8n key` / API key files as secrets that must never be tracked. The repo `.gitignore` already excludes `*sk-proj*`, `*n8n key*`, `*.env`, `.env`, `*secret*`, `*credential*`, `*token*`, `*.pem` — keep these patterns intact.
- When introducing any new secret-like file, confirm it is matched by `.gitignore` (or add a pattern) BEFORE committing, and verify with `git status` / `git diff --cached --name-only`.
- Never log sensitive data (keys, tokens, raw credentials). Scrub or omit them from logs and error messages.
- Validate user-uploaded files before processing: enforce expected type/extension and a size limit, and wrap parsing in graceful error handling so a malformed upload shows a friendly message instead of crashing.
- Do not echo secret values into the UI, terminal output, or committed test fixtures.

---
> Source: [MyProEye-UG/MyProSole](https://github.com/MyProEye-UG/MyProSole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
