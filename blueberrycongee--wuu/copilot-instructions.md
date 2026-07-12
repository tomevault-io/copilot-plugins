## wuu

> - Must complete tasks end-to-end without asking the user for confirmation on small or medium-sized implementation decisions.

## Execution Autonomy

- Must complete tasks end-to-end without asking the user for confirmation on small or medium-sized implementation decisions.
- Must make local naming, refactoring, implementation, and routine workflow decisions independently when they are reversible and do not change security, architecture, or product scope.
- Must create atomic commits during multi-step work as each independent step is completed and verified. Must not bundle unrelated steps into one final commit.
- Must apply existing knowledge, tacit engineering knowledge, and established best practices directly. Must not ask about standard implementation details that a competent engineer should already know.
- Must not interrupt execution for trivial questions, obvious choices, or routine best-practice decisions. Ask only when the choice is irreversible or materially affects security, architecture, or product behavior.

## Product Stage and Development Bias

- This project is in a high-velocity product iteration stage. Optimize for quickly shipping coherent user-facing behavior, not for preserving existing implementation details by default.
- Product intent from the user is the primary source of truth. Existing behavior is evidence, not authority; if the current behavior conflicts with the intended user experience, change the behavior directly.
- Default to developing directly on the current `main` branch in this repository. Do not create a new branch, worktree, or detached work area unless the user explicitly asks for one, the user asks for parallel isolated work, or a concrete safety reason requires isolation.
- Work in small, atomic steps on `main`: each independent behavior change should be implemented, verified, and committed separately. Do not accumulate unrelated changes for one large final commit.
- Prefer decisive product fixes over narrow patches that only silence the immediate symptom. If the root problem is a mismatched product model, fix the model rather than adding local guardrails around the symptom.
- Avoid unrelated changes, not necessary product changes. If the intended behavior requires changing a broader product model, do that directly and keep unrelated refactors out of the commit.
- Keep engineering discipline proportional to risk: inspect the relevant code first, preserve data safety, avoid avoidable regressions, and verify the actual running product path before claiming completion.
- When validation matters, verify against the real app or runtime the user is using, not only an isolated worktree, stale build, or inferred code path.

## Core and Shell Architecture

- The **Go core** (`internal/`, `cmd/wuu/`) is the reusable foundation: agent runtime, providers, tool loop, sessions, config, and the `wuu app-server` subprocess. The core is not coupled to any specific shell.
- The **current shell** is the Electron desktop in `desktop/`. It spawns the core as a subprocess and owns the UI, native integrations, and the IPC bridge.
- **Future shells** (VS Code extension, JetBrains plugin, etc.) consume the core by spawning `wuu app-server`. They do not need to import or fork the Go core; they reuse it as a process.
- A change is **shell-level** when it touches `desktop/` only (UI, native APIs, packaging). A change is **core-level** when it touches `internal/` or `cmd/wuu/`. Keep this boundary clean: do not let the shell leak into the core, and do not let the core depend on Electron APIs.

## Agent-Friendly Text Entrypoint

- Wuu has no TUI. Use Electron for human interaction and `wuu exec` for agents, scripts, CI, and automation.
- When an agent modifies Wuu's agent-facing runtime, verify the product path with `wuu exec --json` or the `wuu debug app-server ...` protocol probes when practical.
- Preserve automation-safe output: default `wuu exec` stdout is only the final answer, JSON mode stdout is only JSONL, and diagnostics belong on stderr.

## Intent First

- Must start from the user's real goal before optimizing local implementation details. The current codebase is context, not the primary definition of what should be built.
- When a request affects product behavior, must reason first about interaction design, visual design, and the broader project vision before choosing detailed code changes.
- Must not assume the existing implementation is correct just because it already exists. Evaluate whether it actually serves the intended user experience and product direction.
- Must not get trapped in minor technical details when they do not materially affect the user's outcome. Prioritize the highest-leverage product and UX decisions first.
- Must still inspect and understand the relevant code before changing it, but that inspection must support the intended outcome rather than let the current implementation define the goal.

## Third-Party Reference Code

- The `thirdparty/` directory contains reference implementations from related agent, CLI, and product codebases. Treat it as a local research library when the user asks for "industry best practices", "how others do this", "reference implementations", or similar guidance.
- When investigating a best-practice question, inspect relevant `thirdparty/` code, docs, and tests with targeted searches before deciding on an implementation. Prefer close analogues over generic assumptions.
- Use third-party code as evidence, not authority. Adapt useful ideas to this repository's existing patterns; do not blindly copy behavior just because another project does it.

## Agent Design Methodology

- When modifying or reviewing core agent behavior, evaluate the design as a closed loop: if an LLM-facing tool is added or changed, verify that prompts teach the model when to use it and when not to use it, and verify that the tool implementation cannot break provider API invariants such as message ordering, tool-call/result pairing, or other protocol rules that would prevent the API from returning a valid response.
- Agent design often contains many thresholds, budgets, scoring weights, stop conditions, retry limits, and other magic numbers. When this repository has no experiment, product constraint, or runtime evidence proving a better value, inspect close analogues in `thirdparty/` and prefer aligning with established defaults before inventing new constants.
- Treat harness comparisons by product fit. Wuu's goal is to become a strong general-purpose BYOK coding-agent harness, so prefer general cross-provider defaults unless there is a clear reason to specialize.
- Reference defaults are evidence, not authority. Diverge when Wuu's product goals, architecture, provider contracts, or user experience require it, and keep the reason explicit.

## User-Facing Output

- Must use plain and common words. Must not use obscure words, inflated wording, or needless jargon.
- Must explain progress and completed work from the product view and the user view whenever possible.
- Must tell the user what was changed, what user problem it addresses, and what experience or behavior is now different.
- Must not focus the explanation on internal code details unless those details are necessary to explain impact, risk, or next steps.
- When summarizing work, prefer user impact and product outcome over implementation trivia.

## Desktop Debug Controls

- Desktop debug UI must not appear in production builds. This includes the run debug button/panel, launch animation preview, development conversation fixtures, style preview toggles, and any future developer-only shortcut buttons.
- In development builds, debug UI must still be hidden by default. Expose it only through the debug controls switch in Settings.
- Future desktop debug buttons or developer-only shortcuts must be gated by the same debug controls setting instead of checking development mode directly.
- The debug controls switch itself must only be visible in development builds. Production builds must not show either the switch or the debug buttons it controls.
- If an e2e test needs the run debug panel, enable it explicitly through the test/build path, and keep production e2e coverage that asserts debug controls are not exposed.

## Local Build and Symlink Refresh

- When the user asks to compile or update the local CLI to the latest source, run `make install` in the repo root.
- Treat `~/.local/bin/wuu -> ~/go/bin/wuu` as the default local path, and refresh the binary at the symlink target. Do not repoint the symlink unless the user explicitly asks.
- After install, verify with:
1. `command -v wuu` and `ls -l ~/.local/bin/wuu ~/go/bin/wuu`
2. `go version -m ~/go/bin/wuu` and confirm `vcs.revision` matches current HEAD
3. `wuu --version` (fallback `wuu version`)
- Explicitly tell the user that running `wuu` now uses the latest local build.

## Release Workflow

- The release source of truth is `docs/release.md` plus `.github/workflows/release.yml`. Keep those in sync when changing the release process.
- Tagged releases are GitHub Actions driven. Do not push tags, publish releases, or upload release assets unless the user explicitly asks for that external action.
- Before creating a release tag, make sure the tag version, root `VERSION`, and `desktop/package.json` version match exactly. The release workflow intentionally fails when they diverge.
- Desktop macOS release artifacts are currently unsigned arm64 preview builds because the project does not yet have Apple Developer ID credentials. Build them in CI on a macOS runner and upload them to the same GitHub Release as the GoReleaser CLI assets.
- Do not describe the current desktop DMG/ZIP as signed, notarized, or public-ready in the Apple Gatekeeper sense. The user-facing release notes and README must mention the unsigned preview status and the `xattr -dr com.apple.quarantine /Applications/wuu.app` workaround for trusted downloads.
- The current desktop release workflow intentionally sets `CSC_IDENTITY_AUTO_DISCOVERY=false` and does not require `MAC_CSC_LINK`, `MAC_CSC_KEY_PASSWORD`, or Apple notarization secrets. Add those secrets and restore `codesign`/`spctl` gates only after the project has Developer ID credentials.
- For desktop release readiness, verify the same product gates the workflow enforces: `cd desktop && npm test`, production debug controls hidden, `npm run release:desktop` from a clean source state or CI, packaged core version has no `dirty` marker, `hdiutil verify`, and `unzip -t`.

## Desktop Dev (`electron-vite dev`) and the `go run` Build Cache

`desktop/`'s dev script (`npm run dev` / `electron-vite dev`) launches the Go
`wuu app-server` as a child process via `go run ./cmd/wuu app-server
--workdir <repo>`. `go run` compiles the binary into Go's build cache
(`~/Library/Caches/go-build/<hash>/wuu`) and execs it. The parent
`go run` process keeps the cache entry alive — the cache survives
across `go run` invocations until that process is killed.

**Trap:** a long-running `electron-vite dev` session keeps a single
`go run` process (and its derived binary) alive for its entire
lifetime. If you change Go source after that, the running binary
**does not pick up the change** — the user sees the old behavior and
will tell you "the fix didn't work" when in fact the fix is correct
but never made it into the running binary. We hit this when the model
reported `ask_user` in its tool list after the ask_user removal
commit: the source was clean, but the cached binary predated the
removal.

Symptoms the user reports contradict the current source:
- A tool the source has removed still appears in the model's tool list
- An error format from the source no longer matches what the model
  sees in tool results
- "I restarted the desktop and it still does X"

When this happens, do not trust your source-level audit alone. The
running binary is the ground truth during a live dev session.

Recovery:
1. Identify the live process chain:
   ```bash
   ps -ef | grep "wuu app-server" | grep -v grep
   ```
   You will see one `go run ./cmd/wuu app-server ...` parent and one
   binary it exec'd, both with the same `--workdir`.
2. Kill both PIDs (kill the `go run` parent first; the child binary
   is its exec target and will exit shortly after, but killing both
   is safe):
   ```bash
   kill <go-run-pid> <binary-pid>
   ```
3. Verify the cache entry is gone (its hash is a function of the
   source state, so a fresh `go run` will compile a new binary even
   if the old one is still on disk):
   ```bash
   ls -la ~/Library/Caches/go-build/*/*-d/wuu 2>/dev/null
   ```
4. Have the user restart desktop (`Ctrl+C` in their dev terminal,
   then `cd desktop && npm run dev`). The new `go run` will compile
   against the current source and the model will see the new behavior.

Quick verification after restart: send `initialize` over the
JSON-RPC stdio of the freshly-spawned app-server and confirm the
returned tool list matches what `tools.New(...).Definitions()`
returns for the current source. (`internal/tools/toolkit.go`
`rebuildRegistry` is the only place tools are registered; if the
list matches there, the running binary is current.)

**A second, deeper trap:** killing the app-server alone is not
enough. `electron-vite dev` runs the Electron main process
(`desktop/src/main/index.ts`) as a separate Node process, and the
main process **is not hot-reloaded by Vite** — only the renderer
and preload are. If you change the main process source (e.g. add
a new `ipcMain.handle("wuu:...", ...)` IPC handler in
`desktop/src/main/index.ts`), the running main process keeps the
old handler table. The renderer's preload gets hot-reloaded to the
new version that invokes the new IPC channel, the invoke fires,
the main process has no handler registered, and the renderer gets
a TypeError / rejected promise. Symptom: the desktop goes to a
white screen the moment something triggers the new IPC.

We hit this with the Settings About build-info panel: the renderer
called `window.wuu.getBuildInfo()`, the main process (started
hours earlier) had no `wuu:build-info` handler, the Settings view
crashed on mount. Killing the stale app-server did not help
because the main process itself was the stale one.

Recovery when this happens: full stack restart, not just
app-server.

```bash
# Find the electron-vite + electron process tree
ps -ef | grep -E "electron-vite|electron\.app/Contents/MacOS/Electron" | grep -v grep

# Kill the whole tree
pkill -f "electron-vite dev"        # node parent
pkill -f "electron-vite"            # any stragglers
pkill -f "/Electron Helper"         # renderer + gpu + utility helpers
pkill -f "go run ./cmd/wuu"         # app-server parent
```

Then have the user run `cd desktop && npm run dev` again. Verify
with `ps -ef | grep -E "electron|wuu" | grep -v grep` that every
relevant process has a fresh start time before claiming the
restart is complete.

---
> Source: [blueberrycongee/wuu](https://github.com/blueberrycongee/wuu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
