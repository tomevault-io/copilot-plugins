## thunk

> Local-first AI coding assistant CLI in Rust. Runtime owns all control flow — model is a stateless text emitter only. Long-term goal: replace Claude Code/Codex with a private self-hosted tool optimized for consumer hardware.

# thunk

Local-first AI coding assistant CLI in Rust. Runtime owns all control flow — model is a stateless text emitter only. Long-term goal: replace Claude Code/Codex with a private self-hosted tool optimized for consumer hardware.

## Hard Stop
Before any commit: `just verify` (fmt --check + check + clippy + test)
Test baseline: 1455 passing via `just verify`
Never make commits — user commits manually.

## Current Phase State
- Phase 32: COMPLETE — TUI overhaul
- Phase 33: COMPLETE — prompt physics, THUNK.md bootstrap, /prompt-physics
- Phase 34: COMPLETE — LSP pre-check, write-then-verify, self-correction, transactions
- Phase 35: COMPLETE — git tool layer, /branch, /commit, /diff, tools/ reorganization
- Phase 36: COMPLETE — .thunk/ migration, abilities, skills, /ability, /skill, prompt physics injection
- Phase 37: COMPLETE — planning, task workflows, /plan, /task, /agent, web fetch, thinking trace
- Phase 38: COMPLETE — vector/embedding layer, retrieval quality logging, /depth, /retrieval
- Phase 39: COMPLETE — code extraction, module reorganization, documentation sync
- Phase 40: COMPLETE — constrained decoding (/constrain), MLX backend, ability compression (/compress)
- Phase 41: COMPLETE — /refactor command, planning turn, EditSequence + sequence executor
- Phase 42: COMPLETE — sequence execution: auto/live signature-change follow-up steps, precise LSP call-site references
- Phase 43: COMPLETE — filesystem watcher, incremental index rebuild on source file changes
- Phase 44: COMPLETE — background save verification surfaced as a passive notification (RebuildFile trigger)
- Phase 45: COMPLETE — MCP integration: dynamic tool registration, `MCPManager` (process lifecycle, stdio transport, config loading), tool calling + surface threading + telemetry wired (Slices 45.1–45.3)
- Phase 46: COMPLETE — personal memory: storage/schema, `MemoryManager` (embedding recall + keyword fallback), /remember, /forget, /memory, /reflect, imperative + reflection filters, MCP polish (escape redirect, no-project-root write scope)
- Phase 47: COMPLETE — tiered shell: `ShellTier` classifier (ReadOnly/FsMutation/Exec), `shell_read` tool, /exec toggle, exec-gate intercept, tier-based NL seeding
- Phase 48: COMPLETE — proactive intelligence: stale-fact scanning, /dnd toggle, proactive config wiring, baseline benchmark run
- Phase 49: COMPLETE — Tauri GUI: `gui` feature, `spawn_backend` extraction, `RuntimeEventDto` event bridge, React + Vite + Tailwind frontend, reset_ok/help DTO discriminators, color token layer (Slices 49.1–49.10)

## Core Principles
- Runtime is the single source of correctness — not the model
- Backend is a stateless text emitter only
- Tools are pure execution units with approval gating
- All reasoning constraints enforced in runtime, not prompt
- Evidence-first retrieval before answer admission
- No text-as-API between subsystems
- Lower layers never depend on higher layers

## Key Files
| Task | File |
|------|------|
| Mutation approval gate | src/tools/registry.rs |
| Shell allowlist | src/runtime/investigation/prompt_analysis.rs |
| Surface enforcement | src/runtime/investigation/tool_surface.rs |
| Evidence gates | src/runtime/investigation/investigation.rs |
| System prompt | src/runtime/protocol/prompt.rs |
| Prompt physics | src/runtime/protocol/prompt_physics.rs |
| Approval stages / transactions | src/tools/pending.rs |
| Turn loop | src/runtime/orchestration/engine.rs |
| Tool dispatch | src/runtime/orchestration/tool_round.rs |
| Runtime config knobs | src/core/config.rs |
| Approval rendering | src/tui/renderer/mod.rs |
| Memory recall/write | src/runtime/memory/manager.rs |
| Memory fact store | src/storage/memory/store.rs |
| Shared types | src/core/ |

## TUI Module Structure
- `src/tui/mod.rs` — terminal setup/teardown and module declarations
- `src/tui/app.rs` — TUI event loop, worker channel integration, render scheduling
- `src/tui/worker.rs` — background `AppContext` command runner
- `src/tui/cursor.rs` — terminal cursor shape/affordance sync
- `src/tui/keybindings.rs` — key event dispatch
- `src/tui/events.rs` — `RuntimeEvent` to `AppState` mapping
- `src/tui/format.rs` — UI formatting helpers
- `src/tui/state.rs` — mutable UI state
- `src/tui/input.rs` — input buffer, history, reverse search, launcher, autocomplete
- `src/tui/collapsible.rs` — pure collapsible summary classification
- `src/tui/commands/mod.rs` — slash command parsing, autocomplete names, launcher entries
- `src/tui/commands/dispatch.rs` — command to `RuntimeRequest`/worker dispatch
- `src/tui/renderer/mod.rs` — renderer, transcript painting, overlays, spinner, approval widget
- `src/tui/renderer/buffer.rs` — cell buffer
- `src/tui/renderer/diff.rs` — frame diff writer
- `src/tui/renderer/style.rs` — `Theme`, colors, packed styles
- `src/tui/renderer/symbols.rs` — symbol interning

Note: `src/tui/renderer/transcript.rs` is not present in the current tree; transcript rendering lives in `renderer/mod.rs`.

## Build
```bash
cargo check --all-targets                                    # fast type-check
cargo test --no-default-features                             # run all tests
cargo build --release --no-default-features                  # build
just verify                                                  # full pre-commit gate
THUNK_TRACE_RUNTIME=1 cargo run --release --no-default-features  # debug
```

## Anti-Patterns — Never Reintroduce
- Parsing assistant text outside tool_codec
- UI-driven execution logic
- Weakening evidence gates
- Model involvement in structural decisions
- Importing AppError or Config from app/ — use core/
- Treating `Theme` as a standalone TUI concern outside `Renderer`

## Reference Docs
@.claude/rules/invariants.md
@.claude/rules/architecture.md
@.claude/rules/slice-discipline.md

## On-Demand Reference — Load Only When Relevant
- `.claude/rules/safe-modification.md` — checklists for adding tools, changing retrieval gates, or changing mutation behavior. Load when making those structural changes.
- `.claude/dev/module-map.md` — module ownership and file locations. Read when adding new modules, tracing ownership boundaries, or unsure where a type lives.
- `.claude/dev/core-loop.md` — runtime loop internals. Read when modifying `engine.rs` or orchestration.
- `.claude/dev/tool-system.md` — tool inventory and wiring. Read when adding or modifying tools.
- `.claude/skills/debug-investigation/` — investigation, guards, failure modes. Read when modifying investigation or candidate selection.
- `.claude/skills/debug-runtime/` — debugging entry points. Read when diagnosing runtime failures.
- `.claude/skills/investigation-planner/SKILL.md` — evidence-first exploration before any implementation. Read before writing any implementation prompt.
- `.claude/dev/tui-keybindings.md` — full keybinding reference. Load when working on TUI input or keybinding handling.

---
> Source: [brendanddev/thunk](https://github.com/brendanddev/thunk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
