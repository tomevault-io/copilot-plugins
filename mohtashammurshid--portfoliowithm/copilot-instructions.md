## portfoliowithm

> This is a Next.js 15 portfolio website (`portfoliowithm`) using App Router with Turbopack.

# AGENTS.md

## Cursor Cloud specific instructions

This is a Next.js 15 portfolio website (`portfoliowithm`) using App Router with Turbopack.

### Services

| Service | Command | Port |
|---------|---------|------|
| Next.js dev server | `npm run dev` | 3000 |

No database, Docker, or external services are required.

### Standard commands

See `package.json` scripts: `dev`, `build`, `start`, `lint`. All are standard Next.js commands.

- **Lint:** `npm run lint`
- **Build:** `npm run build`
- **Dev server:** `npm run dev` (uses Turbopack)

### Environment variables

- `GITHUB_TOKEN` (optional) — enables the GitHub pinned-repos section via GraphQL API. Without it, the section shows a loading/empty state but the app still runs fine.

### Notes

- The dev server uses Turbopack (`next dev --turbopack`) for fast HMR.
- No automated test suite exists in this project; `npm run lint` is the primary code-quality check.
- The app fetches data from GitHub API and npm registry API at request time; these degrade gracefully without network access.

---
> Source: [MohtashamMurshid/portfoliowithm](https://github.com/MohtashamMurshid/portfoliowithm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
