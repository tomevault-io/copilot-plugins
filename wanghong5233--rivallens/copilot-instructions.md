## env-secrets

> Env and secret safety for RivalLens.


# Env and Secret Safety

- Real local env files must not be committed.
- Example env files may contain variable names, comments, placeholders, and safe defaults only.
- Doubao endpoint ids (`ep-...`) are treated as sensitive project credentials and must not appear in docs/code with real values.
- Public env prefixes are browser-visible.
- Do not log full JWTs, API keys, cookies, or `Authorization` headers.
- Do not invent auth bypass headers.
- Avoid duplicate defaults where Docker Compose silently overrides local env.
- Before commit/push, run secret scanning on staged changes or tracked files.

---
> Source: [wanghong5233/RivalLens](https://github.com/wanghong5233/RivalLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
