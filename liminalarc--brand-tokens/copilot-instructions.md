## brand-tokens

> A small, framework-agnostic **design-token package**. Single source of truth for

# brand-tokens — Claude Code Rules

A small, framework-agnostic **design-token package**. Single source of truth for
color / typography / radius / shadow, consumed by multiple apps. Keep it small,
boring, and dependable.

## What this is (and isn't)

- **Is:** values (`tokens.json`) + a build that emits CSS/SCSS/JS for any app to consume.
- **Isn't:** a component library, a framework, or an app. No UI components here (a
  shared component CSS layer is a possible *future* spec, not the core).
- Tokens are **not secrets** — they are already visible in any consuming app's
  public CSS. This repo can be public.

## Golden rules

- **Edit `tokens.json` only.** Never hand-edit anything in `dist/` — it is generated.
- **Rebuild after editing:** `npm run build` (regenerates `dist/`). `dist/` is
  **committed** so git-dependency consumers need no build step.
- **Token naming:** `--sb-<name>` for colors, `--sb-radius-<name>`, `--sb-shadow-<name>`,
  `--sb-font-*`, `--sb-type-<step>-{size,weight}`. Keep names stable — renaming a
  token is a **breaking (major)** change.
- **Light + dark parity:** every color token defines both `light` and `dark`. Never
  add a color with only one.
- **Conventional Commits:** `feat:` (token added), `fix:` (value corrected),
  `chore:`/`docs:`/`refactor:`. Small, logical commits.

## Release workflow

1. Edit `tokens.json` → `npm run build` → review `dist/` diff.
2. Update `CHANGELOG.md`.
3. `npm version <patch|minor|major>` (semver: patch = value tweak, minor = token
   added, mat = token renamed/removed) → `git push --follow-tags`.
4. Cut a GitHub Release. Consuming apps' Dependabot/Renovate raise adoption PRs.

## Source of truth

- **`SPECIFICATIONS.md`** — every change/idea is a spec; update status continuously;
  archive DONE specs. This is a `/flow` project — run `/flow`, `/flow --add`, etc.
- **`DESIGN-SYSTEM.md`** — the human design reference (palette rationale, type
  scale, component conventions). Keep in sync when tokens change.
- **`README.md`** — consume/maintain instructions for app authors.

## Build

- Node 18+. Zero runtime/build dependencies (`build.mjs` uses only `node:fs`).
- Validate before pushing: `npm run build` must succeed and produce a clean `dist/` diff.

---
> Source: [liminalarc/brand-tokens](https://github.com/liminalarc/brand-tokens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
