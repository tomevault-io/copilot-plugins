## werk

> An operative instrument for structural dynamics practice.

# werk

An operative instrument for structural dynamics practice.

## Context

Read `designs/werk-conceptual-foundation.md` for the conceptual architecture — the sacred core, the four frameworks, the vocabulary, and the design decisions. That document is the authority; everything else derives from it.

Run `cargo run --bin werk -- tree` to see the current tension structure.

Run `cargo run --bin werk -- show <id>` for details on any tension.

## Three Interface Surfaces

- **TUI** — `cargo run --bin werk` (or `werk` if installed). The primary experience. A session.
- **CLI** — `cargo run --bin werk -- <command>`. Every gesture as a command. Sessionless.
- **MCP** — `cargo run --bin werk -- mcp`. Protocol surface for AI agents. Stdio transport, 35 tools, direct library calls.
- **Web** — `cargo run --bin werk -- serve`. Axum server at http://localhost:3749. Serves HTML frontend + REST API. Not a WASM app — do not use `trunk`.
- **Desktop** — `cd werk-app && cargo tauri dev`. Tauri app. Requires `cargo install tauri-cli`.

## The Four Frameworks

1. **Architecture of Space** — dimensions, positions, limits, the one spatial law (desired above actual)
2. **Grammar of Action** — gesture primitives, state machine, key bindings per state
3. **Calculus of Time** — two user-set primitives (deadline, order), six computed properties, two recorded facts
4. **Logic of Framing** — what's visible and actionable given context (envelope, zoom, thresholds)

## CLI Conventions

- **`--json` on every command.** All commands support `--json` for structured output. Agents should always use this.
- **`--help` with examples.** Every command has 2-3 usage examples in `--help`. Agents pattern-match off these.
- **`--help` grouped by framework.** Run `werk --help` to see commands organized by Structure, Action, Time, Framing, System.
- **Non-interactive.** No command blocks on stdin or opens `$EDITOR` without a TTY. `stats --health --repair` accepts `--yes`. `reality`/`desire` require explicit value when no TTY detected.
- **Reading surface: 5 commands.** `show <id>` (one tension), `list` (query engine with rich filters), `tree` (hierarchy), `stats` (field aggregates), `log` (logbase — epoch history, cross-tension timeline, ghost geometry, provenance). Old commands (`survey`, `diff`, `ground`, `insights`, `trajectory`, `health`, `context`) still exist but are being consolidated.
- **`--dry-run` on destructive commands.** `resolve`, `rm`, `move`, `split`, `merge` all support `--dry-run` for preview without mutation.
- **Structural gestures.** `split <id> "desire 1" "desire 2"` divides a tension with provenance tracking. `merge <id1> <id2> --into <id>` (asymmetric) or `--as "new desire"` (symmetric) combines tensions. Both create typed edges and cross-tension epochs.
- **Deep addressing.** `#42~e3` (epoch 3), `#42.n3` (note 3), `#42@2026-03` (temporal lookup), `g:ULID` (gesture). Used by `werk log` and the command palette.
- **Edges as substrate.** All structural relationships (parent-child, split provenance, merge provenance) are typed directed edges in the `edges` table, loaded into the FNX DiGraph with edge attributes. The `parent_id` column is maintained for backward compat but edges are the source of truth for new relationships.
- **Note defaults to add.** `werk note 42 "text"` works without the explicit `add` subcommand.
- **Short codes everywhere.** Use `#42` not ULIDs. Short codes are the user-facing addressing scheme.

## CLI Output Design Principles

When modifying command output, follow these principles:

1. **Think from first principles** — why does someone invoke this command? What do they need to know?
2. **Information hierarchy** — identity first (desired above actual), then structural position (parent with context), then temporal situation, then signals by exception.
3. **No redundancy** — never repeat what's already visible. Activity log summarizes mutations ("reality updated") instead of dumping full text.
4. **Most recent first** for temporal data. What just happened matters most.
5. **Inherited context with honest attribution** — show parent's deadline but label it clearly ("none (parent #10 due 2026-05)").
6. **Compact layout** — multiple co-read facts on one line (Status + Created, Position + Last act).
7. **Shared conventions** — use `display_id_named()` for parent context, `format_timestamp()` for consistent times, `format_mutation_summary()` for concise mutation display. All in `werk-shared`.

## Structural Invariants

Before changing data model, display order, or signal logic, check `designs/werk-conceptual-foundation.md`. The sacred core is defined there — desired above actual, theory of closure, signal by exception, gesture as unit of change, locality, structure determines behavior. If a change would violate one of these, stop and discuss.

Everything else (glyphs, colors, chrome, display breakpoints) is changeable.

## Formal Specifications (Quint)

Executable specs of the sacred core live in `specs/`. Written in [Quint](https://quint-lang.org) — a specification language with TLA+ foundations and familiar syntax.

- **Typecheck**: `quint typecheck specs/werk.qnt`
- **Verify core invariant**: `quint run specs/werk.qnt --main=werk --max-samples=10000 --invariant=systemInvariant --backend=typescript`
- **Check strong invariant** (finds containment violations): `quint run specs/werk.qnt --main=werk --max-samples=5000 --invariant=strongInvariant --backend=typescript`
- **REPL**: `quint -r specs/werk.qnt::werk`

Modules: `types.qnt` (domain types), `tension.qnt` (lifecycle state machine), `forest.qnt` (graph/tree invariants), `temporal.qnt` (urgency, containment, sequencing), `gestures.qnt` (gesture atomicity, split/merge), `concurrency.qnt` (MVCC two-writer model), `werk.qnt` (composition + step relation).

**When changing the data model or invariants**, update the Quint specs first, verify they typecheck, then change the Rust code. The specs are the executable version of the conceptual foundation.

Agents: `quint-verifier` (run invariants + witnesses), `quint-analyzer` (plan spec updates from foundation changes).
Commands: `/spec:next`, `/verify:check`, `/verify:witnesses`.
Guideline: `.claude/guidelines/quint-constraints.md` for Quint language limitations.

MCP: The `quint-lsp` MCP server provides LSP diagnostics for `.qnt` files via `.mcp.json`.

## Code Quality

- **UBS (Ultimate Bug Scanner)** runs automatically on every file write/edit via Claude Code hook. Critical findings are surfaced inline.
- Run `ubs --only=rust .` for a full project scan. Run `ubs --only=rust --diff .` for changed files only.
- Fix all CRITICAL findings before committing. Warnings are advisory.
- The database uses **fsqlite** (FrankenSQLite) with MVCC concurrent writers enabled (`PRAGMA fsqlite.concurrent_mode=ON`). Multiple processes can safely write to the same store simultaneously — conflicts are detected at the page level and retried with exponential backoff. The old advisory file lock has been removed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bierlingm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
