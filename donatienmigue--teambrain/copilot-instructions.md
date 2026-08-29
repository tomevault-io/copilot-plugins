## teambrain

> TeamBrain is a git-native, cross-vendor shared memory for AI coding agents:

# TeamBrain — project instructions

TeamBrain is a git-native, cross-vendor shared memory for AI coding agents:
a brain repo of markdown memories + a local daemon (MCP server + index) +
capture hooks + a CI distiller that proposes new memories as pull requests.

## Read first
- docs/internal/CONTRACTS.md — frozen schemas & interfaces. NEVER change these
  without being explicitly asked; if a task seems to require it, STOP and ask.
- docs/internal/BUILD_PLAN.md — the task you are on, with acceptance criteria.

## Non-negotiable principles (tie-breakers for every decision)
1. Git is the source of truth. SQLite index is a rebuildable cache — index
   corruption must never lose data. Anything durable is a file in the repo.
2. Graceful degradation. Hooks fire-and-forget; if the daemon is down, events
   drop silently and the agent session is unaffected. No hook path may block,
   throw to the caller, or exceed 20ms p95.
3. Privacy by construction. Default capture level is `metadata`: never record
   raw prompts, file contents, or diff bodies. Redaction runs before anything
   touches the spool. Never log memory bodies or event payloads at info level.
4. Human-approved writes only. No code path writes to memories/ on main;
   automation writes to branches and opens PRs.
5. Vendor-neutral. All agent-facing behavior goes through MCP + thin per-tool
   hooks. No richer code path for one vendor.
6. Boring dependencies. Adding a dependency requires a one-line justification
   in the commit body. Prefer: node stdlib > small vetted lib > large framework.
   Banned without discussion: ORMs, DI frameworks, langchain-style wrappers.

## Stack (fixed)
TypeScript strict, Node >= 20, pnpm workspaces monorepo. ESM only.
better-sqlite3 + FTS5 + sqlite-vec; fastembed (bge-small, local ONNX);
official @modelcontextprotocol/sdk; commander for CLI; vitest for tests;
zod for all runtime validation at boundaries. No default exports.

## Commands
pnpm install · pnpm build · pnpm test · pnpm lint · pnpm bench
pnpm test:integration (spins fixture repos under /tmp)

## Testing rules
- Every package has unit tests; every schema in docs/internal/CONTRACTS.md has a zod schema
  with fixture-based round-trip tests.
- Tests never touch the network. LLM calls go through the Provider interface;
  tests use the FakeProvider with recorded fixtures in testdata/.
- Negative tests are first-class: retired memories absent from retrieval;
  user-scope files absent from any pushed tree; redaction corpus is release-gating.
- Performance budgets are tests: pnpm bench fails if retrieval p95 > 300ms on
  the 5k-memory fixture or hook handler > 20ms p95.

## Definition of done (per task)
Code + tests + acceptance commands from docs/internal/BUILD_PLAN.md pass + short
entry in docs/internal/DEVLOG.md (what/why/tradeoffs, 5 lines max). If an
acceptance criterion cannot be met, do not redefine it — report the blocker.

## Forbidden
- Editing docs/internal/CONTRACTS.md schemas (ask first).
- `any` types at package boundaries; skipping zod validation on external input.
- Writing to the user's real ~/.Codex or ~/.cursor in tests (use TMPDIR fakes).
- console.log in library code (use the shared logger, packages/core/log).
- Silent catch blocks. Degradation must be logged at debug level with a reason.

---
> Source: [donatienmigue/TeamBrain](https://github.com/donatienmigue/TeamBrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
