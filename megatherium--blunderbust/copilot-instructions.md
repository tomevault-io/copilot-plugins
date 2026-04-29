## blunderbust

> This project uses **bd (beads)** for issue tracking.

# Agent Instructions
## Issue Tracking

This project uses **bd (beads)** for issue tracking.
Run `bd prime` for workflow context (MANDATORY!), or install hooks (`bd hooks install`) for auto-injection.

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

If there's any contradiction: `bd prime` is right. AGENTS.md is not 100% up to date.

## Landing the Plane (Session Completion)

**When ending a work session** before sayind "done" or "complete", you MUST complete ALL steps below.
Work is NOT complete until `git push` succeeds.
Push is not allowed until the work is REVIEWED

**MANDATORY WORKFLOW:**
State A:
  1. **File issues for remaining work** - Create issues for anything that needs follow-up
  2. **Run quality gates** (if code changed) - Tests, linters, builds
  3. **Run CODE REVIEW & REFINEMENT PROTOCOL** - See `bd prime` for details
-- DO NOT CROSS THE LINE BY TOURSELF --
State B (after SOMEONE ELSE has reviewed it):
  4. **Update issue status** - Close finished work, update in-progress items
  5. **PUSH TO REMOTE** - This is MANDATORY:
    ```bash
    git pull --rebase
    git add (careful with using -A, the user sometimes leaves untracked crap lying around) && git commit ...
    git push
    git status  # MUST show "up to date with origin"
    ```
  6. **Clean up** - Clear stashes, prune remote branches
  7. **Verify** - All changes committed AND pushed
  8. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- Pushing is not allowed until the work is successfully reviewed
- If push fails, resolve and retry until it succeeds
- Failure to follow this order creates double commits (one for code, one for .beads/issues.jsonl)
- MODIFY ticket, git STAGE/PUSH - else you will be creating extra commits

## Lessons learned

- Be aware of Go's pass-by semantics especially with closures.
- Don't assume you know what a function does by its name alone. The devil is in the details.
- In Bubble Tea, never mutate application state (like maps or UI models) inside a `tea.Cmd` background goroutine; always return a `tea.Msg` and mutate state safely within the main `Update()` thread.
- Never use defer cancel() on a context passed to Dial if the returned connection will use that context after the function returns - always use a separate dial context with its own timeout.
- teatest strips ANSI color codes, making it fundamentally incapable of testing visual focus states - navigation tests belong in agent_tui_test.go where websocket streaming preserves ANSI codes.
- Delete tests that simulate actions but only verify trivial assertions - vacuous tests like assert.True(t, len(out) > 0) provide false confidence and should be removed entirely.
- WaitFor is for observable state changes, not for timing delays - use time.Sleep for rapid key sequences where intermediate states don't produce detectable output differences.
- When the reviewer says "this is completely untouched," stop and actually look at the exact line they're pointing to
- Adding visible text indicators to UI (like ▶ for focus) is valuable for users even when your testing framework can't leverage them - don't conflate UI improvements with testability.
- A 3.0/10 review score means you fundamentally misunderstood the requirements - don't try to justify partial fixes, just implement exactly what the reviewer specified.
- For harness/process validation, binary matching is alias-based (1 harness can map to multiple executable names, e.g. `kilo` and `kilocode`).
- If the UI isn't updating properly: are the caches being dirtied properly?

## Modern tooling

All kinds of modern replacements for standard shell tools are available: rg, fd, sd, choose, hck
The interface is nicer for humans. You pick whatever feels right for you.

### Vibe MCP

If the `vibe_run` tool is available to you think of it as a not so bright but fast agent. Use it for low effort but content intensive tasks like tracing flow, finding code, etc.
- Run it with agent=auto-approve so it uses all its tools
- max_turns = 5 by default, so give it more if the request has length to it

## File Editing Strategy

- **Use the Right Tool for the Job**: For any non-trivial file modifications, you **must** use the advanced editing tools provided by the MCP server.
  - **Simple Edits**: Use `sed` or `write_file` only for simple, unambiguous, single-line changes or whole-file creation.
  - **Complex Edits**: For multi-line changes, refactoring, or context-aware modifications, use `edit_file` (or equivalent diff-based tool) to minimize regression risks.

## Commit Messages

- **Conventional Commits**: All commit messages **must** adhere to the Conventional Commits specification.
  - **Format**: `<type>[optional scope]: <description>`
  - **Example**: `feat(harvester): implement reverse-scroll logic for Gemini`
  - **Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`
- **Beads extra**: Add a line like "Affected ticket(s): bb-foo", can be multiple with e.g. review tickets
- **WARNING**: Forgetting the ticket reference line is a commit message format violation. Double-check before committing.

## Documentation

- **New Features**: When implementing new features, **must** update documentation:
  - User-facing features: Update README.md with usage examples
  - Template context changes: Document new fields and legacy compatibility behavior
  - Behavioral changes: Update AGENTS.md to inform agents
  - Always keep both files in sync

## Reference Material (Mandatory When Relevant)

Before working in these areas, you MUST read the corresponding reference file:

- **TUI Testing** → `agent-docs/tui_testing_guide.md`
  - Modifying or testing TUI components (`internal/ui/`)
  - Writing or debugging `agent-tui` tests
  - Making visual changes that require color/focus state verification

- **Build & Run** → `agent-docs/build_and_run.md`
  - Building the project (make/build commands)
  - Running bdb locally
  - Configuring CI/CD pipelines
  - Adding new CLI flags or exit codes

- **Dolt Internals** → `agent-docs/dolt_internals.md`
  - Modifying `internal/data/dolt/`
  - Working on ticket store implementations
  - Debugging database connection issues

## File Picker Recents

The file picker (accessed via `p` key) uses a bubbles fork with recents support.

### Architecture

- **bubbles fork**: `internal/ui/filepicker/filepicker.go`
  - `MaxRecents` field controls maximum recents (default:5)
  - `RecentsChangedMsg` emitted when recents list changes
  - `ShowRecents` enables dual-pane UI (filepicker + recents)

### Configuration Fields

- **TUI-specific config** (separate from `domain.Config`):
  - `config.TUIConfig.FilePickerMaxRecents` - Maximum recent directories
  - `config.TUIConfig.FilePickerRecents` - Recent directories list
  - YAML: `filepicker_max_recents` and `filepicker_recents` in `tui_config.yaml`

### Key Implementation Points

1. **Loading**: In `NewUIModel()` and `Init()` (model.go), TUI config is loaded:
   - `config.LoadTUIConfig(app.Opts.TUIConfigPath)` reads from `tui_config.yaml`
   - `fp.Recents` is populated from loaded config
   - `fp.MaxRecents` is set from loaded config (defaults to 5)

2. **Saving**: In `handleProjectMsgs()` (model.go), `filepicker.RecentsChangedMsg`:
   - Loads current TUI config
   - Updates `cfg.FilePickerRecents` with new recents
   - Saves config immediately using `config.SaveTUIConfig()`
   - Logs errors to stderr

3. **UI**: Dual-pane view in `RenderFilePicker()` (view_filepicker.go):
   - Left: Directory browser
   - Right: Recents list (when `ShowRecents=true`)
   - `Tab` key swaps focus between panes

### Config File Locations

- CLI resolution (see `cmd/blunderbust/root_run.go:resolveTUIConfigPath()`):
  1. `~/.config/blunderbust/tui_config.yaml` (if exists)
  2. `./tui_config.yaml` (fallback)

### Testing

See `internal/config/yaml_tui_test.go`:
- `TestLoadTUIConfig_WithValues` - Full config loading
- `TestLoadTUIConfig_DefaultMaxRecents` - Default value handling
- `TestSaveTUIConfig` - Persistence verification
- `TestTUIConfig_RoundTrip` - Full save/load cycle

---
> Source: [Megatherium/blunderbust](https://github.com/Megatherium/blunderbust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
