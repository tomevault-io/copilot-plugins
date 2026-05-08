## gitforge

> Core rules and context for the gitforge CLI + profile-site template


# gitforge Project Rules & Context

- This repo is both:
  - A **Vite + React + TS** template for a GitHub profile site.
  - An **npm CLI package** named `gitforge` that generates static data from the GitHub API.

## CLI & Config

- The single source of truth for config is `gitforge.config.json` (ignored by git).
- Keep `gitforge.config.example.json` as the only tracked config template.
- The generator script is `scripts/generate-github-data.js` and is exposed as the `gitforge` binary via `package.json`.
- Preferred usage patterns to keep in mind:
  - `gitforge [owner] [--type user|org]`
  - Or: read `gitforge.config.json` when no CLI args are given.

## Generated Files

- Treat these as **generated**; do not hand-edit or rely on them for long-term docs:
  - `src/generated/githubData.ts`
  - `src/siteContent.json`
- When editing content for the site, prefer working against `src/siteContent.json` (it is regenerated, so keep that in mind when suggesting workflows).

## App Constraints

- Use **functional React components**, **TypeScript**, and **simple CSS** (no Tailwind / UI libraries).
- The page is single-page (no routing) and should stay:
  - Clean, dark, minimal, developer-focused.
  - Deterministic and static (no runtime GitHub calls from the browser).

---
> Source: [amide-init/gitfolio](https://github.com/amide-init/gitfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
