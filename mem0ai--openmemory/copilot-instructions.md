## openmemory

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`openmemory-cli` (CLI command: `openmemory`) — a hub-and-spoke CLI + TUI that ports **conversations** between AI coding harnesses (Claude Code, Codex, OpenCode). v1 fidelity is **text turns only**; tool calls, tool results, and thinking blocks are dropped at import and counted, not carried.

All code lives under `cli/`. Runtime is **Bun** (not Node) — everything runs `.ts`/`.tsx` directly, no build step.

**Distribution is source-checkout only**: `scripts/install.sh` (curl-able) clones this repo, runs `bun install`, and drops a shim on PATH. There are deliberately **no binary releases, no npm publish, and no Homebrew formula** for now — don't add release/publish machinery without asking. Consequence: anything committed under `cli/src/` is public source (e.g. a PostHog project API key is fine — write-only — but real secrets never).

Put any generated markdown, plans, or TODO files under `claude-generated-docs/`.
Put any blog/article generated files in `blog-posts/`.

## Commands

Run from `cli/`:

```bash
bun install
bun test                                   # full suite (~90s; includes real round-trip e2e)
bun test test/detect.test.ts               # single file
bun test -t "some test name"               # by name filter
bun run typecheck                          # tsc --noEmit
bun run cli <command>                      # = bun run src/cli.ts
```

CLI subcommands (`src/cli.ts`): `detect`, `port --from <h> --to <h> (--all | --id <sid>...) [--force] [--json]`, `verify [--from] [--to a,b] [--sample N | --all]`, `tui`. There is deliberately **no** standalone `import`/`export` subcommand — in-memory Hub IR can't be passed between CLI invocations (dropped in S8).

### Sandbox commands (test harness)

Run from `cli/`. All target `~/.openmemory-sandbox/` (`OPENMEMORY_SANDBOX_ROOT` overrides) and never touch real stores unless a backup ran first. `--harness claude-code|codex|opencode|all` (default `all`).

- `bun run sandbox:backup --harness all` — snapshot real stores to timestamped backups.
- `bun run sandbox:restore --harness all [--from <timestamp>]` — restore latest/named backup.
- `eval "$(bun run sandbox:reroute --harness codex)"` — point the porter CLI at the sandbox (write mode; includes `OPENMEMORY_LEDGER_PATH` isolation).
- `eval "$(bun run sandbox:reroute --harness codex --app)"` — env for launching the desktop app against the sandbox (best-guess pending spike).
- `bun run sandbox:seed --projects 3 --convos 5 [--source-root <dir>]` — seed sandbox claude store from real convos.
- `bun run sandbox:clear --harness all` — wipe sandboxed conversations (sandbox paths only, guarded).

Verification helpers (modules, not CLIs): `scripts/sandbox/store-check.ts` (`sessionExists` via destination `discover()`), `scripts/sandbox/resume-probe.ts` (non-interactive resume: `claude --resume <id> -p`, `codex exec resume <id>`, `opencode run -s <id>`), `scripts/sandbox/issues.ts` (`logIssue`/`readIssues` → `issues.jsonl`). The full e2e flow is orchestrated by the `port-sandbox-test` skill — pure reporter, failures logged, fixes dispatched after the run.

## Architecture

The whole tool is a **hub-and-spoke** around one intermediate representation. Every import produces a `PortableSession`; every export consumes one. Harnesses never talk to each other directly.

- `src/ir.ts` — **the hub.** `PortableSession` Zod schema. `sourceSessionId` is the idempotency key; `cwd` is authoritative (never re-derive it from encoded path names); `sourceMetadata` is a lossless bag; `droppedTurns` counts tool/thinking-only turns skipped at import.
- `src/adapter.ts` — the contract each spoke implements. `ImportAdapter` = harness → IR (`detect`/`discover`/`parse`). `ExportAdapter` = IR → harness (`detect`/`write`/`teardown`).
- `src/adapters/{claude-code,codex,opencode}/` — one import + one export module per harness. `src/adapters/index.ts`'s `registerAllAdapters()` is the **only** place adapters touch the registry; adapters never register themselves.
- `src/registry.ts` — register/lookup adapters by harness + direction. `listInstalled()` intersects S7 detection with what's actually registered.
- `src/detect.ts` (S7) — pure, read-only, never-throws probing of which harnesses are present and read/writable. Takes an injectable `DetectEnv` so tests can stub `HOME`/`CODEX_HOME`/`PATH`/`XDG_DATA_HOME`.
- `src/core/orchestrator.ts` (S8) — the engine: detect → import → export pipeline over the registry, emitting a `PorterEvent` stream. **Zero TUI dependency by construction.** Per-session failures are isolated (emit `error`, never abort the batch).
- `src/core/ledger.ts` (S8) — **the single idempotency authority for the whole tool.** Keyed on the triple `(sourceHarness, sourceSessionId, destHarness)`. JSON file under `$XDG_DATA_HOME/openmemory/`. Writes are serialized per path via an in-process lock.
- `src/tui/` — SolidJS + `@opentui` terminal UI (not Ink). `app.tsx` is the 5-step wizard shell (Destination → Source → Select → Import → Done); `wizard.ts` holds all render-free, unit-testable logic; `screens/` are thin views over it.

## Decision wiki (`docs/brain/`)

Curated, citeable knowledge that travels with the repo. `sources/` holds dumped articles/notes; `decisions/YYYY-MM-DD-*.md` records decisions, each `[[linking]]` its sources; `index.md` lists every page. Cite by path+commit. This is distinct from mem0 (auto-captured episodic memory).

**Write-delegation rule:** write pages inline when the content is already in context (a decision we just made, a short note). Delegate to a **fork** only when the entry requires digesting bulky source material first (a long dumped article, multi-file synthesis) — the fork reads and returns the distilled page, keeping raw material out of the main context. Never delegate a small write for "speed": the write is cheap, the agent isn't.

## Conventions & gotchas

- **Idempotency lives only in `core/ledger.ts`.** Adapters have NO ledger of their own: `write()` always writes, `teardown()` always removes. The orchestrator decides skip/create/update-in-place by consulting the ledger, then passes `WriteOptions` (`force`, `existingDestId`) into `write()`. Don't reintroduce per-adapter dedupe state (three incompatible ones were deleted in the S8 revision).
- **Provenance, not a side-channel.** Because adapters own no ledger, each stamps provenance onto the destination itself (e.g. a `openmemorySource` field / `session.metadata`) so `teardown`/update-in-place can find "our" rows by scanning.
- **`yieldToEventLoop()` in the orchestrator loops is load-bearing.** `parse()`/`write()`/ledger calls are synchronous disk I/O wrapped in `async` — their `await` resolves via a microtask, not real I/O, so without an explicit yield the whole batch starves the TUI render loop. Keep it.
- **Test isolation via config injection, never real stores:** `setProjectsRoot()` (claude), `configureCodexExport()` / `CODEX_HOME` (codex), `OPENCODE_DB` + `OPENCODE_DISABLE_CHANNEL_DB=1` (opencode), `setLedgerPath()`. The `verify` command redirects every destination at a fresh `mktemp` root unconditionally — it never writes to a real `~/.claude`, `~/.codex`, or the real opencode DB.
- **opencode DB landmine:** prefer the real `opencode.db` over a dev-clone `opencode-local.db`; set `OPENCODE_DISABLE_CHANNEL_DB=1` to target the installed DB. The export happy path shells out to `opencode import` (with child cwd = `session.cwd`); direct `bun:sqlite` writes are the CLI-absent fallback.
- **Worktree cwd is remapped at the hub entry.** `core/worktree.ts`'s `createCwdRemapper()` (per-cwd memoized) runs in `orchestrator.importAll()` right after `parse()`, rewriting a git-worktree `cwd` to its parent project so *everything* downstream — the TUI Select grouping, project counts, and all three exports — sees the real repo, not a Conductor worktree (`nairobi`/`banjul`/…). `exportSessions()` re-runs it as an idempotent safety net (the `worktreeSource` stamp short-circuits) for sessions built outside `importAll`. Resolution order: live worktree via `git rev-parse` (any git worktree) → deleted worktree via Conductor's `conductor.db` (`workspace_path`→`repos.root_path`, survives deletion) → else leave `cwd`. The original path is stamped into `sourceMetadata.originalCwd` + `worktreeSource`. Default-on; set `OPENMEMORY_KEEP_WORKTREE_CWD=1` to disable, `OPENMEMORY_CONDUCTOR_DB` to point the fallback at a test db. Because it mutates `cwd` in one place, all consumers inherit it with zero adapter/screen changes — don't add per-adapter or per-screen worktree logic.
- **Known v1 fidelity gaps** (asserted in `verify`): codex export doesn't serialize `session.title`; `--all` round-trips surface a handful of opencode text mismatches (reported, not fatal).
- Self-documenting code over comments for new code, but note this codebase carries deliberately long explanatory header comments on the tricky modules — match that when touching them.

---
> Source: [mem0ai/openmemory](https://github.com/mem0ai/openmemory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
