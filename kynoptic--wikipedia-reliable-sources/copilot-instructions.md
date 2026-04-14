## wikipedia-reliable-sources

> Apply file and module structure conventions across the codebase.

# Code structure and clarity guidelines

- Group code by feature domain (e.g. `/auth`, `/orders`), not by file type.
- Avoid deep directory nesting; favor flatter, clearer structures.
- Start each service or app with a single, clear entry point (e.g. `main.ts`, `index.js`).
- Delete unused or legacy code proactively.
- Use descriptive, intention-revealing names over comments or cleverness.
- Encapsulate side effects (e.g. DB, HTTP, filesystem) within boundary modules.
- Fail loudly in development; log or assert on unexpected states.
- Tag incomplete logic clearly with `// TODO:` or `// FIXME:` annotations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kynoptic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
