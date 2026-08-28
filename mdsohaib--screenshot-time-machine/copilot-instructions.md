## screenshot-time-machine

> `screenshot-time-machine` (`stm`) is a small Node CLI: find the localhost app → discover pages → full-page screenshot each → dated folder + `manifest.json` → report which pages changed. Thousands of people will read this source. Optimize for readability, one runtime dependency, and excellent error messages.

# AGENTS.md: how to work in this repo (humans and AI agents)

`screenshot-time-machine` (`stm`) is a small Node CLI: find the localhost app → discover pages → full-page screenshot each → dated folder + `manifest.json` → report which pages changed. Thousands of people will read this source. Optimize for readability, one runtime dependency, and excellent error messages.

## Setup (5 commands)

```bash
git clone https://github.com/mdsohaib/screenshot-time-machine && cd screenshot-time-machine
npm ci
npm test                    # node:test: unit + CLI + e2e against the built-in fixture site
npm run fixture             # keeps the fixture site up on 127.0.0.1:<port> for manual runs
node bin/stm.js --url http://127.0.0.1:<port>
```

`npm run check` = TypeScript over JSDoc (`checkJs`, strict). `npm run lint` = Biome. Both run in CI.

## Module map (v0.1)

| File | Owns |
|---|---|
| `bin/stm.js` | Node-version guard, EPIPE guard, `process.exitCode = await main()` |
| `src/cli.js` | `util.parseArgs`, subcommand dispatch, human summary strings |
| `src/index.js` | `run(options)` / `list(options)`, the programmatic API the tests call |
| `src/detect.js` | localhost port probing, "several apps" resolution |
| `src/discover.js` | URL normalization, link filtering, sitemap parsing (regex, no XML dep) |
| `src/capture.js` | browser bootstrap (cached shell → chrome → msedge → install), per-page pipeline, worker pool, overlay handling |
| `src/store.js` | snapshot folder naming, slug, incremental manifest, sha256 "changed", gitignore append |
| `src/list.js` | `stm list` |
| `src/types.d.ts` | `Manifest`, `PageResult`, `RunOptions` |
| `skills/stm/SKILL.md` | the agent skill (`stm skill` prints it) |
| `test/fixture/site.js` | the one fixture site (unit, e2e, CI smoke, GIF, benchmark) |

## Hard rules (a PR that breaks one will be closed)

1. **One runtime dependency** (`playwright-core`, exact-pinned). No new dependency for XML, colors, args, tables, spinners. Use `util.parseArgs`, `util.styleText`, regex.
2. **No new flags or commands without an issue first.** The answer is usually no. A good-first-issue never adds a flag.
3. **stdout is only the summary (human) or one JSON document (`--json`).** Progress goes to stderr, and only when stderr is a TTY. `--json` prints ≤ 2 stderr lines.
4. **A failing page never aborts a run.** Record it, continue, exit 2.
5. **stm never deletes anything.** It writes only inside `--out` and may append one line to an existing `.gitignore` (announced).
6. **No page-derived text** (titles, headings, body) in `--json` or the manifest. That output is read by AI agents, so it is a prompt-injection channel. Errors are truncated to 200 chars.
7. **No config file, no daemon/watch mode, no telemetry, no hooks or MCP servers in the plugin.**
8. Readability beats line count. Never golf. Boring over clever. `execFile` only, never `shell: true`.
9. Vocabulary in user-facing strings: a folder is a "snapshot", a PNG is a "screenshot", URLs are "pages", the target is "your app". Never "routes", "dev server", "timeout" in human mode.
10. **You must understand your change.** Using AI tools is fine. Submitting generated output you haven't reviewed and can't explain is not. Say in the PR if an agent wrote most of it.

## Before you open a PR

- `npm test && npm run check && npm run lint` are green locally.
- Paste the real `stm` summary line from a run against the fixture (or your app) in the PR description.
- If you touched `capture.js`, run the fixture determinism test twice (`changed: []` on the second run) and say so.
- One-line entry under `## [Unreleased]` in `CHANGELOG.md`.
- If you said no to something on purpose, add a row to `DECISIONS.md`.

---
> Source: [mdsohaib/screenshot-time-machine](https://github.com/mdsohaib/screenshot-time-machine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
