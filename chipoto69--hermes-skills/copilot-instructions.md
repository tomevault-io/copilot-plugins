## hermes-skills

> This repository is a public Hermes skill catalog. Treat `meta.ts` as the catalog truth, `skills/*/SKILL.md` as installable artifacts, and `README.md` as the public page.

# Agent Operating Notes

This repository is a public Hermes skill catalog. Treat `meta.ts` as the catalog truth, `skills/*/SKILL.md` as installable artifacts, and `README.md` as the public page.

## Rules

- Keep every skill self-contained under `skills/<name>/`.
- Preserve provenance for generated or vendored work via `GENERATION.md` or `SYNC.md`.
- Do not put private notes, secrets, local paths, or unredacted source dumps into public skills.
- Do not publish upstream PRs to third-party repos without explicit user approval.
- Use short operational skill bodies. Put bulk reference material in `references/`.
- Run validation before committing.

## Commands

```bash
pnpm install
pnpm catalog list
pnpm catalog check
pnpm validate
pnpm lint
```

Create a manual skill skeleton:

```bash
pnpm catalog new my-skill "Use when ..."
```

Report unregistered skill folders:

```bash
pnpm catalog cleanup
```

## Catalog model

- `skills`: installable Hermes skill artifacts.
- `sources`: upstream docs/code repos used to generate new skills.
- `vendors`: external skill packs synced into this catalog.
- `instructions`: maintainer runbooks for adding skills and source lanes.

If a script changes generated files, commit the generated output too. Consumers install from GitHub; they should not need to run generation first.

---
> Source: [chipoto69/hermes-skills](https://github.com/chipoto69/hermes-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
