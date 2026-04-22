## summarize

> - Hard rule: single source of truth = `~/Projects/summarize`; never commit in `vendor/summarize` (treat it as a read-only checkout).

# Summarize Guardrails

- Hard rule: single source of truth = `~/Projects/summarize`; never commit in `vendor/summarize` (treat it as a read-only checkout).
- Note: multiple agents often work in this folder. If you see files/changes you do not recognize, ignore them and list them at the end.

## Workspace layout (note)

- Monorepo (pnpm workspace).
- Packages:
  - `@steipete/summarize` = CLI + UX (TTY/progress/streaming). Depends on core.
  - `@steipete/summarize-core` (`packages/core`) = library surface for programmatic use (Sweetistics etc). No CLI entrypoints.
- Versioning: lockstep versions; publish order: core first, then CLI (`scripts/release.sh` / `RELEASING.md`).
- Dev:
  - Build: `pnpm -s build` (builds core first)
  - Gate: `pnpm -s check`
  - Import from apps: prefer `@steipete/summarize-core` to avoid pulling CLI-only deps.
- Daemon: restart with `pnpm -s summarize daemon restart`; verify via `pnpm -s summarize daemon status`.
- Rebuild (extension + daemon): run **both** in order:
  1. `pnpm -C apps/chrome-extension build`
  2. `pnpm summarize daemon restart`
- Extension tests:
  - `pnpm -C apps/chrome-extension test:chrome` = supported automated path.
  - Firefox Playwright extension tests are not reliable (`moz-extension://` limitation); default `test:firefox` skips.
  - Use `pnpm -C apps/chrome-extension test:firefox:force` only for explicit diagnostics.
- Commits: use `committer "type: message" <files...>` (Conventional Commits).

---
> Source: [steipete/summarize](https://github.com/steipete/summarize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
