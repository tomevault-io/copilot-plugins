## ulm

> A unified CLI gateway to Claude, Codex, Gemini, and Ollama. See [`docs/prds/2026-05-23-ulm.md`](./docs/prds/2026-05-23-ulm.md) for the product brief.

# CLAUDE.md — ulm

A unified CLI gateway to Claude, Codex, Gemini, and Ollama. See [`docs/prds/2026-05-23-ulm.md`](./docs/prds/2026-05-23-ulm.md) for the product brief.

## Repo state

- **Pre-code.** PRD is promoted (`docs/prds/2026-05-23-ulm.md`). All 10 features scaffolded under `docs/features/` with SPEC, TASKS, SCORE, DECISIONS; F-04 also has DBSCHEMA, and F-06 / F-08 / F-10 have INTERFACE-CONTRACTS. All SPECs have been critiqued and ACs populated.
- **Implementation language: Node.js + TypeScript** (PRD §13 Q-06, resolved 2026-05-23, post-promotion). Supersedes the earlier "Resolved → Rust" reading. `engines.node: ^20.10 || ^22 || ^24`. Cascades into G-06 install footprint (≤ 2 MB tarball, ≤ 8 MB installed — not a static-binary cap), F-09 release-packaging (npm-native primary), and all toolchain references.
- Working directory: Windows. Bash and PowerShell both available.
- All 36 `/vskit:*` commands are wired as Claude Code slash commands under `.claude/commands/vskit/`.

## Setup notes

- **Stop hook depends on `jq`.** The `worklog-stop-hook.sh` shells out to `jq` on every turn (parses the Stop event JSON, line 29). Without it on PATH, `docs/worklog/YYYY-MM-DD.md` files will not be created. On Windows: `winget install jqlang.jq` then restart Claude Code so the hook subprocess inherits the new PATH.
- **Node ≥ 20.10 required** once source code lands. Install via nvm/volta/fnm; pin via `.nvmrc` (or `engines.node` in `package.json`).

## Commands

Framework: docs/bundle/vertical-slices-ai-framework.md

### Repo-Specific Overrides

| Command | Shell invocation |
|---------|-----------------|
| `/vskit:run-tests` | `npm test` (vitest unit + integration) — populated once `package.json` lands |
| `/vskit:run-e2e` | `npm run test:e2e` (node-pty harness suite) — populated once `tests/e2e/` lands |
| `/vskit:health-check` | N/A — `ulm` is an interactive CLI, no health endpoint |
| `/vskit:deploy` | Tag-triggered: `git tag v$VERSION && git push --tags` → GitHub Actions runs `.github/workflows/release.yml` (`npm publish --provenance` per F-09 AC-10) |

> **Do not redefine** `/vskit:check deploy`, `/vskit:score`, `/vskit:critique spec`, `/vskit:critique prd`,
> `/vskit:audit-traceability`, `/vskit:init-framework`, or any other framework-owned command.
> Overriding them silently changes ship-gate semantics. (spec §9)

### Framework settings

- **Weekly token budget:** 100000 — default per spec §5.3 (bold format required so `worklog-stop-hook.sh` line 193 regex matches; backticked form is silently ignored).

### Toolchain (per PRD §14.2, post-Node.js retrofit)

| Tool | Floor | Purpose |
|------|-------|---------|
| Node.js | 20.10 | Runtime |
| npm | 10.2 | Package manager + provenance publishing |
| TypeScript | 5.4 | Source language; compiled to ESM |
| esbuild | latest | Bundle `src/**/*.ts` → `bin/ulm.mjs` |
| vitest | 1.x | Unit + integration + snapshot tests |
| tinybench | 2.x | Bench harness (§7.1 perf gates) |
| node-pty | 1.x | PTY-driven REPL integration tests |
| nock | 13.x | HTTP mocking (Ollama transport tests) |
| `@iarna/toml` | 3.x | state.toml read/write (F-04 D-01) |
| `proper-lockfile` | 4.x | Cross-platform file locking (F-04 D-05) |
| `commander` | 12.x | CLI flag parsing (F-07 D-04) |
| `fastest-levenshtein` | 1.x | Unknown-command suggestion (F-06 D-01) |
| `tree-kill` | 1.x | Cross-platform subprocess-tree signaling (F-02 D-11) |
| `license-checker-rseidelsohn` | 4.x | GPL/AGPL transitive-license gate (F-09 D-04) |

---
> Source: [rafaelmelo007/ulm](https://github.com/rafaelmelo007/ulm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
