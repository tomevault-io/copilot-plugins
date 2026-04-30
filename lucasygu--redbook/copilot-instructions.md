## redbook

> **NEVER publish npm packages directly from local.** Always use the `npm-publish-cli` agent to bump version and publish via CI. The GitHub Actions workflow (`Publish to npm`) handles publishing on push to main. Publishing locally races CI and causes workflow failures.

# CLAUDE.md — redbook CLI

## Publishing

**NEVER publish npm packages directly from local.** Always use the `npm-publish-cli` agent to bump version and publish via CI. The GitHub Actions workflow (`Publish to npm`) handles publishing on push to main. Publishing locally races CI and causes workflow failures.

**Dual publishing (automated):**
- **npm** — publishes when `package.json` version changes on push to main
- **ClawHub** — publishes when `SKILL.md` changes on push to main (uses `CLAWHUB_TOKEN` secret)

Both are conditional jobs in `.github/workflows/npm-publish.yml`. Manual trigger via `workflow_dispatch` is also available.

## Build & Test

- `npm run build` — TypeScript compile + chmod
- No test suite yet — verify manually with `redbook whoami`

## Project Structure

- `src/cli.ts` — CLI entry point, 21 commands, `getClient()` is the single cookie→client funnel
- `src/lib/client.ts` — XHS API client (`postComment`, `replyComment`, etc.)
- `src/lib/cookies.ts` — Cookie extraction with Chrome profile auto-discovery
- `src/lib/signing.ts` — Request signing
- `src/lib/analyze.ts` — Viral note analysis and question detection
- `src/lib/reply-strategy.ts` — Batch reply filtering, templating, rate-limited execution
- `src/lib/template.ts` — Viral content template extraction
- `src/lib/health.ts` — Note health check: level detection, sensitive words, tag count
- `src/lib/render.ts` — Card rendering (markdown → PNG via puppeteer-core, optional dep)
- `SKILL.md` — Skill documentation (serves both Claude Code and OpenClaw/ClawHub)
- `.github/workflows/npm-publish.yml` — CI: build, npm publish, ClawHub publish

## Render Command (Optional)

- `redbook render <file>` — markdown → styled PNG cards using user's existing Chrome
- Requires `puppeteer-core` and `marked` (optional peer deps, lazy imported)
- Uses Chrome at `/Applications/Google Chrome.app/...` or `CHROME_PATH` env var
- No cookies/XHS API needed — purely offline
- HTML/CSS templates embedded as TypeScript template literals (no external files)

## Cookie Architecture

- Uses `@steipete/sweet-cookie` to read browser cookies
- Auto-discovers Chrome profiles via `~/Library/Application Support/Google/Chrome/Local State`
- Keychain timeout patched to 30s in node_modules (upstream bug: hardcoded 3s)
- `--chrome-profile` flag available as escape hatch, but auto-discovery handles most cases

---
> Source: [lucasygu/redbook](https://github.com/lucasygu/redbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
