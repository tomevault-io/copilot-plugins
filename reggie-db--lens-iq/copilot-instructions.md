## prefer-bun

> Use bun instead of npm for local package management and scripts


# Prefer bun over npm (locally)

This repo uses `bun` as its local package manager. The lockfile is
`bun.lock` (there is no `package-lock.json`). `npm install` is slow here,
so default to `bun` for every local command.

## Command mapping

| Instead of | Run |
| --- | --- |
| `npm install` | `bun install` |
| `npm install <pkg>` | `bun add <pkg>` |
| `npm install -D <pkg>` | `bun add -d <pkg>` |
| `npm uninstall <pkg>` | `bun remove <pkg>` |
| `npm ci` | `bun install --frozen-lockfile` |
| `npm run <script>` | `bun run <script>` |
| `npx <tool>` | `bunx <tool>` |

## Rules

- Never run `npm install` / `npm ci` locally. Use `bun install`.
- Do not create or commit `package-lock.json`. Keep `bun.lock` as the only
  lockfile.
- When adding deps, let `bun add` update `package.json` + `bun.lock`.

## Exception: the Databricks Apps platform

Do NOT touch the npm-based deploy path. The Apps platform runs `npm ci` and
`npm run build` at deploy time (that's why `.npmrc` sets `production=false`).
That is intentional, leave `package.json` scripts npm-compatible and don't
swap them to bun-only invocations.

---
> Source: [reggie-db/lens-iq](https://github.com/reggie-db/lens-iq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
