## milkdrop-extension

> Workspace instructions for coding agents working on this repository.

# AGENTS.md

Workspace instructions for coding agents working on this repository.

## Scope

- Project type: GNOME Shell extension in GJS with a GTK4 renderer subprocess and a native OpenGL helper.
- Primary goal: keep shell-side code robust and non-blocking while preserving visual/audio parity behavior.
- Canonical high-level docs:
	- `README.md`
	- `CLAUDE.md`
	- `docs/architecture.md`
	- `docs/development.md`

## Quick Start Commands

Run all commands from the repository root.

- Build helper:
	- `meson setup build`
	- `meson compile -C build`
- Unit/integration tests:
	- `gjs -m tests/run.js`
- Parity tests:
	- `gjs -m tests/run-parity.js`
- Benchmarks:
	- `gjs -m tests/bench/run.js`
	- `gjs -m tests/bench/run.js -- --json`
- Local development via just:
	- `just install`
	- `just reinstall`
	- `just nested`
	- `just renderer`
	- `just logs`

## Mandatory Constraints

- Runtime is GJS, not Node.js. Do not add npm/webpack/transpilation assumptions.
- Do not use `eval()` or `new Function()` in expression code.
- Shell process safety first: code under `src/extension/` runs in GNOME Shell PID.
- Keep IPC non-blocking; respect bounded queues and backpressure behavior.
- Treat GNOME 47, 48, and 49 compatibility as explicit when touching shell internals.
- Prefer minimal, targeted changes; avoid unrelated refactors.

## Architecture Boundaries

- `src/extension/`: shell-side lifecycle, monitor orchestration, audio capture, evaluator, presets, IPC server.
- `src/renderer/`: GTK4 renderer process, `GtkGLArea`, GL bridge/client.
- `src/renderer/gl-helper.c`: native OpenGL helper and readback path.
- `src/shared/`: protocol/shared constants.

Before changing cross-process behavior, review:

- `docs/architecture.md`
- `src/shared/ipc-protocol.js`
- `src/extension/ipc.js`
- `src/renderer/ipc-client.js`

## Testing Expectations

- Any change in `src/extension/expr/`, evaluator, parser, or preset loading should run:
	- `gjs -m tests/run.js`
	- `gjs -m tests/run-parity.js`
- Any performance-sensitive or parsing-path change should also run:
	- `gjs -m tests/bench/run.js`
- Any change touching `src/renderer/gl-helper.c` should rebuild with Meson before tests.

## Known Pitfalls

- GJS introspection APIs may return tuples for getters (for example GStreamer structure integer getters).
- Preset handling must remain crash-resilient; avoid weakening probe/quarantine safeguards.
- GLib idle/timeout callbacks can race with teardown; guard against stale object references.
- Bench and parity tooling assumes repository-root cwd.

## Link-First Reference Index

- Architecture and process model: `docs/architecture.md`
- Development/debug workflows: `docs/development.md`
- Expression semantics: `docs/milkdrop-expression-spec.md`
- Parity progress and context: `docs/parity-validation-progress.md`, `docs/projectm-parity.md`
- Known decisions and open questions: `QUESTIONS.md`
- Refactor guardrails: `REFACTOR_CHECKLIST.md`

## Agent Working Style

- Prefer read-only exploration first, then smallest safe patch.
- Preserve existing module boundaries and naming unless change requires otherwise.
- Include concise rationale in commit/patch summaries: what changed, why, and which tests were run.
- If unsure about process ownership (shell vs renderer vs helper), stop and verify before editing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mauriciobc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mauriciobc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
