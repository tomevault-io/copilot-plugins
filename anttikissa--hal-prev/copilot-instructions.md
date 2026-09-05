## hal-prev

> Small but capable Bun + TypeScript coding agent with multi-tab TUI, persistent sessions, and file-backed IPC.

# HAL

Small but capable Bun + TypeScript coding agent with multi-tab TUI, persistent sessions, and file-backed IPC.

Current code lives in `src/`; previous version is in `old-src/` for reference.

## Rules

- After every code change to `src/`, run `bun scripts/cloc.ts` and include the output.
- Use red-green TDD.
- To test, run ./test, never `bun test`
- Use Bun, never node.js. `bunx` - no `npx`. No build step.
- State is stored under `HAL_STATE_DIR` (default: `$HAL_DIR/state`).
- Secrets live in `$HAL_DIR/auth.ason` (gitignored). Non-secrets in `$HAL_DIR/config.ason`.
- Conversation events are append-only: `state/sessions/<id>/history.asonl`.
- Tabs are real sessions; `/compact` rotates session history.
- Prefer tabs for indentation (width 4).
- When editing, collapse consecutive blank lines to one. Files should end with a newline.
- After completing a task, commit to git.
- Commit each finished task with a concise, capitalized imperative subject in sentence case (for example `Add feature X` or `Rename zot to bar`) with no trailing period.
- If asked to learn something, write it to `AGENTS.md` so I can remember it the next time.
- If learning requires editing new code, edit the code and ask user to restart.
- `[todo] <text>` — append the text as a bullet to `TODO.md` and commit. No questions, no hesitation.
- `/bug <description>` — captures terminal snapshot + debug log. You can paste images and text. Use it to self-debug UI issues.
- Keep code MINIMAL. No migration code unless asked! Screw backwards compatibility. But warn when stuff is going to break.
- Don't invent flags/presets/options that nobody will use.
- I'm on a laptop. It doesn't have Home and End keys for example.
- Prefer one working mechanism over layered fallbacks/proof-of-concept additions; only add another path when the first is clearly insufficient.
- Don't reimplement what the system already provides. For example, `open('wx')` is an atomic exclusive-create — no need to build a `mkdir`-based mutex on top of it.
- Doing a big task? 1. Read files and think. 2. Plan - write it to file (docs/plans). 3. Implement.
- Non-test runtime code budget: max 10k LOC. `bun run cloc` to check.

## Git safety

- **Never `git commit --amend`** — make a new commit instead. Amend rewrites history and is dangerous when other sessions may have committed on top.
- **Never `git reset`** — it rewrites history. If you need to undo your own commit, use `git revert`.
- **Always `git log --oneline -5` before committing** — verify HEAD is what you expect. Another session may have pushed commits.
- **One commit per logical change, no squashing** — don't try to "clean up" commit history. Messy but correct beats clean but destructive.

## User interface guidelines

Hal is plumbing-visible by default: don't try to hide complexity, file paths, or the like.

Bad: "[system] system prompt reloaded (file changed)"
Good: "[system] reloaded SYSTEM.md (file changed)" or "(model changed)"

Bad: "[promoted] this process is now the owner"
Good: "[promoted] pid 12345 is now the owner"

Insert `<blink />` (50ms pause) or `<blink ms="400" />` (custom duration) in streamed text for comedic timing, dramatic pauses, or musical phrasing. The tag is stripped from output and converted to a delay.

## TUI

- Canonical TUI behavior doc: `docs/tui.md` (update it with any TUI rendering/input behavior change).

- Avoid prompt/status flicker: keep terminal redraws in a single full-frame write.
- On kitty/ghostty-compatible TTYs, wrap frame writes with synchronized output (`\x1b[?2026h` ... `\x1b[?2026l`).
- If adding another frame write path beyond `render()`, factor the sync wrapper into shared helper/constants and use it everywhere frames are emitted.

## Testing

- **NEVER** use `bun --eval` or `bun -e` — Use `bunx tsgo --noEmit` for syntax/type checks.
- Write tests for new features and bug fixes
- Command for TDD loop: `bun test --test-name-pattern='<feature>'`
- Before committing, run tests: `./test` (parallel runner; `bun test` is sequential and slow)
- Unit tests live alongside code, e2e tests in `src/tests/`
- `src/cli/test-driver.ts` — lightweight harness for testing prompt/keybinding behavior. Uses `TestDriver` class: type chars, send keys, assert text/cursor/selection. See existing tests in `test-driver.test.ts`.

## SYSTEM.md preprocessor

`SYSTEM.md` is preprocessed before being sent to the model (`src/runtime/system-prompt.ts`):
- `${model}`, `${cwd}`, `${date}`, `${session_dir}`, `${eval}` are replaced with runtime values.
- `::: if model="glob"` ... `:::` fenced blocks conditionally include content by model name.
- HTML comments are stripped.
- Consecutive blank lines are collapsed.
- `.asonl` files (conversation, session, IPC logs) use ASONL format — parse with `ason.parseAll()`, not line-by-line JSON.

## Hot patchability

- Prefer this pattern for runtime modules:
	- `function foo() { ... }`
	- `export const moduleName = { foo, ... }`
- Cross-module calls should go through namespace objects (`moduleName.foo(...)`), so eval-time monkey patches take effect immediately.
- Keep direct exports for types and constants.
- `runtime.ts` is class-based; patch via `runtimeCore.getRuntime()` or `Runtime.prototype`.
- Eval scripts can import with `~src/` and receive `ctx.runtime`.
- If a module-level value is a runtime tuning knob (threshold, timeout, size limit, default window), put it in a mutable exported config object and read it at call time; keep plain `const` only for true constants like regexes, paths, URLs, and schemas. See `src/session/prune.ts` `pruneConfig` for an example.

## TODOs

See `TODO.md`.

## Startup performance

**Target: <100ms to interactive** (time from process start to user being able to type).

Startup flow: `run` → `main.ts` (owner election) → `startRuntime()` → `publish()` → CLI connects → render → interactive. See `src/runtime/startup.ts` and `src/perf/startup-trace.ts` for the trace points.

Key guidance:
- **Minimize work before `publish()`** — that's the gate for CLI connection. Session restore is the minimum; everything else should be deferred.
- **Avoid sequential async operations** on the startup path. Parallelize independent work (e.g. per-session scans via `Promise.all`).
- **Defer non-critical work** (interrupted tool resolution, pending-turn scanning, continuations) to after publish. Fire-and-forget with `void (async () => { ... })()`.
- **Read history once** per session and reuse it — don't call `readHistory`/`loadAllHistory` multiple times for different checks on the same session.

## Architecture (high level)

- `run` — bash entry point. Restart loop (exit 100 = restart), env setup, `-f`/`-s` flag handling.
- `main.ts` — owner election + CLI startup.
- `src/ipc.ts` — file-backed IPC bus (`state/ipc/`). See `docs/ipc.md`.
- `src/runtime/*` — owner runtime (scheduling, commands, agent loop).
- `src/cli/*` — TUI + CLI client. See `docs/tui.md`.
- `src/session/*` — session persistence, messages, compaction, rotation. See `docs/session.md`.
- `src/session/prune.ts` — context pruning (strip old tool results, images, thinking). See `docs/context-management.md`.
- `src/providers/*` — provider adapters (Anthropic, OpenAI).
- `src/utils/ason.ts` — ASON serialization.

---
> Source: [anttikissa/hal-prev](https://github.com/anttikissa/hal-prev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
