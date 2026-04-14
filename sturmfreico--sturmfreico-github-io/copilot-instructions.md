## sturmfreico-github-io

> Sturmfrei company website — a static marketing site built with **Astro v5**, **Tailwind CSS**, and **TypeScript**. No backend services, databases, or external dependencies required for local development.

# AGENTS.md

## Cursor Cloud specific instructions

### Project overview

Sturmfrei company website — a static marketing site built with **Astro v5**, **Tailwind CSS**, and **TypeScript**. No backend services, databases, or external dependencies required for local development.

### Available commands

See the `scripts` section in `package.json` and the README for the full list. Key commands:

| Command | Purpose |
|---|---|
| `npm run dev` | Start dev server on port 4321 (with `--host` for network access) |
| `npm run build` | Production build to `./dist/` |
| `npm run lint:eslint` | Run ESLint |
| `npm run format` | Format with Prettier |

### Caveats

- **Pre-existing lint error:** `Hero2.astro` has an unused `Background` import that triggers an ESLint error. This is in the existing codebase and should not block work.
- **Tailwind config warning:** During build, a warning appears about `tailwind.config.cjs` and ES module loading. This is cosmetic and does not affect the build output.
- **No `.env` required:** The site runs fully without environment variables.
- **Content:** Blog posts are MDX files in `src/content/post/`. The CMS (Decap CMS) requires external git-gateway auth and is not needed for local development.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sturmfreico) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
