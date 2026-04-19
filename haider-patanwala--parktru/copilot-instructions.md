## parktru

> Follow `AGENTS.md` as the canonical project rule file.

Follow `AGENTS.md` as the canonical project rule file.

Key constraints:

- Inspect the repo before changing it.
- Reuse existing `src/server/eden.ts`, MongoDB, and Better Auth helpers.
- Keep `src/app` thin and move feature logic into `src/features/<feature>/...`.
- Use feature-first MVC.
- Do not invent Zero APIs if the package is not installed.
- Keep UI modern, clean, and low-cognitive-load.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haider-patanwala) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
