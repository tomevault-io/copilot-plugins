## deadskills

> `deadskills` is an open-source CLI (`npx deadskills`) that parses local agent session transcripts and reports per-skill usage: invocation counts, token cost, and "dead" skills (installed but never invoked — pure context tax). Claude Code and Codex supported out of the box. Full strategy, competitive analysis, and launch playbook: **PROJECT.md** (read it first for any product/roadmap question).

# deadskills — project context for Claude

## What this is

`deadskills` is an open-source CLI (`npx deadskills`) that parses local agent session transcripts and reports per-skill usage: invocation counts, token cost, and "dead" skills (installed but never invoked — pure context tax). Claude Code and Codex supported out of the box. Full strategy, competitive analysis, and launch playbook: **PROJECT.md** (read it first for any product/roadmap question).

## Stack

TypeScript 5 (strict, ESM-only), Node ≥18, **zero runtime dependencies** (deliberate — a context-bloat auditor with a dependency tree is a punchline; output is hand-rolled ANSI in `src/output/format.ts`, args hand-parsed in `src/cli.ts`). Dev deps only: tsup (build), vitest (tests), typescript, @types/node.

## Commands (Makefile is the entry point)

- `make install` / `make build` / `make test` / `make typecheck`
- `make check` — typecheck + tests, run before any commit
- `make run` — build + run report against this machine's real agent data
- `make dead` / `make json` — other CLI modes
- `make publish-dry` — verify package before npm publish
- CLI itself: `deadskills [dead|doctor] [--json] [--agent X] [--claude-dir P] [--codex-dir P] [--project P]`

## Architecture

- `src/adapters/` — one file per agent implements `AgentAdapter` (see `types.ts`): `detect()`, `loadEvents() → {events, health}`, `skillRoots()`. Registry in `index.ts` — new agent = one file + one registry line. `claude-code.ts` and `codex.ts` are the reference implementations. All parsing MUST fail soft AND record skips in `ParseHealth` — silent undercounting is a bug (`doctor` command surfaces drift).
- `src/discovery/skills.ts` — enumerates SKILL.md files under adapter-provided roots, parses frontmatter, estimates token costs.
- `src/analysis/report.ts` — joins events × skills into `Report` (includes parseHealth). `schema/report.schema.json` is the versioned canonical output (CC0 — it's the strategic asset; additive changes only, breaking = bump schemaVersion). `--json` emits an **array** of Reports (one per agent).
- `src/output/format.ts` — ANSI formatters (report/dead/doctor); screenshot-worthiness of the report is the growth strategy.
- `src/cli.ts` — entry; has an entrypoint guard so tests can import `parseArgs`.
- `tests/fixtures/{claude,codex}/` — hand-written anonymized fixtures (never commit real transcripts).

## Transcript format notes (validate before trusting)

- Claude Code: `~/.claude/projects/<encoded-path>/*.jsonl`; skill markers = `tool_use` named "Skill" + `<command-name>` text markers.
- Codex: `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`, lines `{timestamp, type, payload}`; skill detection is HEURISTIC (SKILL.md path in function_call args, `$mention` in user messages) — needs validation against real rollouts.

## Current state (2026-07-14)

**v0.1 feature-complete.** Zero-dep rewrite, Claude Code + Codex adapters, zombie tier (`status: active|zombie|dead`, 90-day cutoff), `--since` window (30d/8w/6m/ISO), fully-qualified-first attribution (bare-name collisions → `ambiguousInvocations`, attributed to none), explicit-vs-auto invocation counts, `doctor` + parse health. 30 tests passing, typecheck clean, single ~22 KB ESM build, verified e2e against fixtures for both agents.

## Next steps (in order)

1. **Validate both parsers against real data** — `make run` on a machine with real `~/.claude` and `~/.codex`; use `deadskills doctor` output to find unrecognized line shapes; fix format assumptions (Codex skill-invocation markers especially — they were written from documented observations + heuristics, not verified).
2. **Reserve the name**: `npm publish` (0.1.0) + create the `deadskills` GitHub repo — name verified free on npm and GitHub 2026-07-14; don't let it get sniped.
3. Repo topics/description per PROJECT.md §2 (SEO lives in description, not name).
4. v0.2 headline: `prune` (reversible archive) + trigger-quality insight. Full roadmap: PROJECT.md §6.
5. Announce in anthropics/claude-code issue #35319 (pre-qualified audience) once v0.1 is on npm.

## Conventions

- Fail-soft parsing everywhere, but every skip must increment ParseHealth — never silently drop.
- Token numbers are estimates (chars/4) labeled with "~", and upper bounds — do not add tokenizer deps for precision.
- Zero runtime deps is a hard rule. New adapters: one file + fixtures + tests (see CONTRIBUTING.md).
- Attribution matches normalized names (`plugin:x` → `x`); known collision risk documented in PROJECT.md brainstorm — fully-qualified-first matching is a welcome improvement.
- No telemetry, no network I/O. Privacy is the moat with this audience.
- Report contains no local paths by design (safe to paste in issues).

---
> Source: [anandsaini18/deadskills](https://github.com/anandsaini18/deadskills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
