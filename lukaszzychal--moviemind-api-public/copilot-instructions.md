## 050-security-baseline

> - Never add/commit secrets. Prefer environment variables. Never log secrets.


# Security baseline

- Never add/commit secrets. Prefer environment variables. Never log secrets.
- Validate input, enforce auth/authorization, and consider rate limiting for public endpoints.
- If output/logs may contain sensitive data: stop and warn.
- For AI-related features: treat user input as untrusted (prompt injection). Keep strict output formats.

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
