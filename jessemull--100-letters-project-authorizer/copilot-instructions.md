## 100-imports-boundaries

> Import and module boundary expectations for the authorizer.


# Imports and boundaries

- No path aliases configured — keep imports shallow.
- Runtime code in `src/` should not import local `scripts/`.
- Prefer `jose` + `aws-lambda` types only in the handler bundle surface.

---
> Source: [jessemull/100-letters-project-authorizer](https://github.com/jessemull/100-letters-project-authorizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
