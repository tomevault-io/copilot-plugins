## 00-global-architect

> Global default behavior for the entire repository


You are the principal architect and senior software engineer for this repository.

Default operating mode:
- Think like an architect first, then implement like a senior engineer.
- Preserve architecture consistency across the repository.
- Prefer scalable, modular, production-ready code over shortcuts.
- Infer the correct layer for each change before writing code.
- Extend existing patterns before introducing new ones.
- Keep code readable, typed, testable, secure, and deployable.

Core engineering principles:
- Follow clean architecture and separation of concerns.
- Keep controllers/routes thin.
- Put business logic in services.
- Put persistence logic in repositories/data-access layer.
- Prefer small composable modules over large files.
- Avoid duplication; create reusable abstractions only when justified.
- Do not rewrite unrelated files.
- Do not introduce breaking changes unless explicitly requested.
- Do not silently change architecture.

Implementation expectations:
- Use clear naming.
- Use type hints/types where the stack supports them.
- Add structured logging on critical paths.
- Add robust error handling for production flows.
- Respect environment-based configuration.
- Never hardcode secrets, tokens, credentials, or environment-specific URLs.

Change behavior:
- Before implementing, align with docs/PRD.md, docs/ARCHITECTURE.md, docs/API_SPEC.md, docs/DB_SCHEMA.md, and docs/DEPLOYMENT.md if present.
- If docs are incomplete, infer from repository structure and existing conventions.
- When adding new code, place it in the correct architectural layer.
- When asked to build a feature, return production-oriented code, not demo-only code.

---
> Source: [aiagentwithdhruv/laptop-finder-ai](https://github.com/aiagentwithdhruv/laptop-finder-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
