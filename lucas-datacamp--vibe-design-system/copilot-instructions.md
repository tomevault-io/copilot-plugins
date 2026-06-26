## design-context

> Load design context from local PRODUCT.md and DESIGN.md before UI work


# Design Context

Your design context lives in **`PRODUCT.md`** and **`DESIGN.md`** at the project root (gitignored).

Before UI or design work:

1. Read `PRODUCT.md` if present — register, users, personality, anti-references, principles.
2. Read `DESIGN.md` if present — colors, typography, components.
3. If `PRODUCT.md` is missing, run `/impeccable init`. Do not infer brand from `README.md` or `prompts/`.
4. Load context: `node .claude/skills/impeccable/scripts/context.mjs` (requires `npx impeccable skills install` first).

Do not hardcode design decisions here. Those local files are the source of truth.

# Before building UI

Do not jump straight to code. Interview first (`/grill-me`):

1. Restate the problem in plain language (what fails, why it matters, what success looks like). No file names or solution language unless essential.
2. Ask one question at a time. Wait for an answer.
3. Do not assume branding from `README.md`, `prompts/`, or an empty `PRODUCT.md`.
4. Stay strategic before colors, fonts, or components.
5. Explore the codebase when you can answer from the repo.
6. Write confirmed answers to `PRODUCT.md` before implementation.

Do not infer taste from repo docs or start coding while the interview is open.

---
> Source: [lucas-datacamp/vibe-design-system](https://github.com/lucas-datacamp/vibe-design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
