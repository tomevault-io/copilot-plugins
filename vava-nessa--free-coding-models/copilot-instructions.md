## free-coding-models

> After completing any feature or fix, the agent MUST:

# Agent Instructions

## Post-Feature Testing

After completing any feature or fix, the agent MUST:

1. Run `pnpm test` to verify all unit tests pass (62 tests across 11 suites)
2. If any test fails, fix the issue immediately
3. Re-run `pnpm test` until all tests pass
4. Run `pnpm start` to verify there are no runtime errors
5. If there are errors, fix them immediately
6. Re-run `pnpm start` until all errors are resolved
7. Only then consider the task complete

This ensures the codebase remains in a working state at all times.

## Release Process (MANDATORY)

When releasing a new version, follow this exact process:

1. **Version Check**: Check if version already exists with `git log --oneline | grep "^[a-f0-9]\+ [0-9]"`
2. **Version Bump**: Update version in `package.json` (e.g., `0.1.16` → `0.1.17`)
3. **Commit ALL Changed Files**: `git add . && git commit -m "0.1.17"`
   - Always commit with just the version number as the message (e.g., "0.1.17")
   - Include ALL modified files in the commit (bin/, src/, test/, README.md, CHANGELOG.md, etc.)
4. **Push**: `git push origin main` — GitHub Actions will auto-publish to npm
5. **Wait for npm Publish":
   ```bash
   for i in $(seq 1 30); do sleep 10; v=$(npm view free-coding-models version 2>/dev/null); echo "Attempt $i: npm version = $v"; if [ "$v" = "0.1.17" ]; then echo "✅ published!"; break; fi; done
   ```
5. **Install and Verify**: `npm install -g free-coding-models@0.1.17`
6. **Test Binary**: `free-coding-models --help` (or any other command to verify it works)
7. **Only when the global npm-installed version works → the release is confirmed**

**Why:** A local `npm install -g .` can mask issues because it symlinks the repo. The real npm package is a tarball built from the `files` field — only a real npm install will catch missing files.

## Real-World npm Verification (MANDATORY for every fix/feature)

**Never trust local-only testing.** `pnpm start` runs from the repo and won't catch missing files in the published package. Always run the full npm verification:

1. Bump version in `package.json` (e.g. `0.1.14` → `0.1.15`)
2. Commit and push to `main` — GitHub Actions auto-publishes to npm
3. Wait for the new version to appear on npm:
   ```bash
   # Poll until npm has the new version
   for i in $(seq 1 30); do sleep 10; v=$(npm view free-coding-models version 2>/dev/null); echo "Attempt $i: npm version = $v"; if [ "$v" = "NEW_VERSION" ]; then echo "✅ published!"; break; fi; done
   ```
4. Install the published version globally:
   ```bash
   npm install -g free-coding-models@NEW_VERSION
   ```
5. Run the global binary and verify it works:
   ```bash
   free-coding-models
   ```
6. Only if the global npm-installed version works → the fix is confirmed

**Why:** A local `npm install -g .` can mask issues because it symlinks the repo. The real npm package is a tarball built from the `files` field — if something is missing there, only a real npm install will catch it.

## Test Architecture

- Tests live in `test/test.js` using Node.js built-in `node:test` + `node:assert` (zero deps)
- Pure logic functions are in `src/utils.js` (extracted from the main CLI for testability)
- The main CLI (`bin/free-coding-models.js`) imports from `src/utils.js`
- If you add new pure logic (calculations, parsing, filtering), add it to `src/utils.js` and write tests
- If you modify existing logic in `src/utils.js`, update the corresponding tests

### What's tested:
- **sources.js data integrity** — model structure, valid tiers, no duplicates, count consistency
- **Core logic** — getAvg, getVerdict, getUptime, filterByTier, sortResults, findBestModel
- **CLI arg parsing** — all flags (--best, --fiable, --opencode, --openclaw, --tier)
- **Package sanity** — package.json fields, bin entry exists, shebang, ESM imports

## GitHub Contributors

When new PRs are merged, add the contributor's GitHub handle to the footer in `bin/free-coding-models.js` (the `Contributors:` line near line 775), separated by spaces. Also update this list:

- @whit3rabbit
- @PhucTruong-ctrl

## Testing the TUI with agent-tui

The project's TUI is built with raw ANSI escape codes + chalk. To visually test TUI behavior, use **agent-tui** — a PTY-based TUI automation tool that lets the AI agent spawn, screenshot, and interact with the terminal app.

### Setup

`agent-tui` is installed as a devDependency (`pnpm add -D agent-tui`). All commands run through `npx agent-tui`.

**One-time:** Start the daemon before any TUI testing session:

```bash
npx agent-tui daemon start
```

### Core Commands

All commands below use `npx agent-tui <command>`. The daemon manages sessions in the background.

| Command | What it does |
|---------|-------------|
| `npx agent-tui run --cols 120 --rows 40 -- node bin/free-coding-models.js` | Spawn the TUI in a virtual terminal |
| `npx agent-tui run -- node /full/path/to/bin/free-coding-models.js --tier S` | Spawn with CLI flags (use full path) |
| `npx agent-tui screenshot` | Capture current screen as text |
| `npx agent-tui screenshot --strip-ansi` | Capture screen without ANSI colors |
| `npx agent-tui screenshot --json` | Capture as JSON (includes session_id) |
| `npx agent-tui press T` | Send a key press |
| `npx agent-tui press ArrowDown ArrowDown Enter` | Send multiple keys |
| `npx agent-tui press Ctrl+C` | Send Ctrl+C |
| `npx agent-tui type "search text"` | Type literal text |
| `npx agent-tui wait "Provider" -t 5000` | Wait for text to appear (5s timeout) |
| `npx agent-tui wait --stable` | Wait for screen to stop changing |
| `npx agent-tui kill` | Kill the current session |
| `npx agent-tui sessions` | List active sessions |
| `npx agent-tui sessions cleanup --all` | Clean up all dead sessions |

### Key Reference

| Key | Action | Use Case |
|-----|--------|----------|
| `T` | Cycle tier filter | Test filtering (All → S+ → S → A+ → A → A- → B+ → B → C → All) |
| `P` | Open Settings screen | Test API key config, enable/disable providers |
| `Z` | Cycle mode | Test mode switching (OpenCode CLI → Desktop → OpenClaw) |
| `R` | Sort by rank | Verify rank-based sorting |
| `Y` | Sort by tier | Verify tier-based sorting |
| `O` | Sort by origin | Verify origin-based sorting |
| `M` | Sort by model name | Verify model name sorting |
| `L` | Sort by latest ping | Verify ping sorting |
| `A` | Sort by avg ping | Verify average ping sorting |
| `S` | Sort by SWE score | Verify SWE score sorting |
| `N` | Sort by context window | Verify context window sorting |
| `H` | Sort by health/condition | Verify health-based sorting |
| `V` | Sort by verdict | Verify verdict sorting |
| `U` | Sort by uptime | Verify uptime sorting |
| `↑`/`↓` | Navigate rows | Move cursor up/down |
| `Enter` | Select model | Choose model |
| `Ctrl+C` | Exit | Quit the TUI |

### Example Test Flow

```bash
# 1. Ensure daemon is running
npx agent-tui daemon start

# 2. Spawn the TUI (use full path)
npx agent-tui run --cols 120 --rows 40 -- node /Users/vava/Documents/GitHub/free-coding-models/bin/free-coding-models.js

# 3. Wait for it to render, then screenshot
npx agent-tui wait --stable
npx agent-tui screenshot --strip-ansi

# 4. Test tier filter — press T, wait, verify
npx agent-tui press T
npx agent-tui wait --stable
npx agent-tui screenshot --strip-ansi

# 5. Test navigation — press Down twice
npx agent-tui press ArrowDown ArrowDown
npx agent-tui wait --stable
npx agent-tui screenshot --strip-ansi

# 6. Clean up
npx agent-tui kill
```

### When Should the Agent Use agent-tui?

Use `agent-tui` when:

- **Visual Testing Needed** — Changes affect TUI rendering, layout, colors, or formatting
- **Interaction Testing** — New keypress handlers, filters, or navigation logic
- **Regression Detection** — Verify existing flows still work after code changes
- **User-Facing Features** — Settings screen, mode switching, tier filtering

**Do NOT use agent-tui** for:
- Unit test verification (use `pnpm test` instead)
- Code-only logic changes (use tests for pure functions)
- Build errors (use `pnpm build:web` or `pnpm start`)

### Tips

- Always `npx agent-tui daemon start` before the first `run` — it's idempotent (no-op if already running)
- Use `--strip-ansi` for screenshots you want to parse/log — removes color codes
- Use `--json` for programmatic parsing of screenshot data
- Use `npx agent-tui wait --stable` after key presses to avoid reading mid-render
- Use `npx agent-tui sessions cleanup --all` if sessions get stuck
- The daemon persists across test runs — no need to restart between sessions

---

## Changelog (MANDATORY)

**⚠️ CRITICAL:** Every new version MUST REPLACE the entire `CHANGELOG.md` file with ONLY the content from THAT specific version. Do not append to a long history.

- Check all commits, code changes, and work done since the last version bumped to ensure the changelog is **complete**.
- Add necessary details, intentions, and explanations to the changelog entries. Provide clear, user-facing explanations of *why* changes were made and *how* they work.
- Use the current version from `package.json` as the single header (e.g. `## [0.1.64] - YYYY-MM-DD`).
- The `CHANGELOG.md` file should ONLY contain this single version's release notes.
- List changes under `### Added`, `### Fixed`, or `### Changed` as appropriate.
- Keep the structure clean so it can be reused directly in the GitHub Release notes screen.
- Update `CHANGELOG.md` BEFORE committing and pushing.

**Why this is critical:** The changelog is used directly for GitHub release notes. It should only contain the comprehensive notes for the current release to avoid publishing the entire history over and over.

## Version Bump Workflow (/bump command)

When user requests `/bump`, `"push commit"`, or `"bump a new version now"`, execute this comprehensive workflow:

### 1. Check Version Status
- Check current version in `package.json`
- Check last published version: `git log --oneline | grep "^[a-f0-9]\+ [0-9]"`
- If multiple uncommitted version bumps exist, consolidate them into the next sequential version

### 2. Update Changelog
- Review all work done since the last published version
- REWRITE the entire `CHANGELOG.md` file to contain ONLY the release notes for the new version
- Include comprehensive details, intentions, and explanations for all changes
- Ensure changelog is user-facing with clear bullet points

### 3. Update Documentation
- Review and update `README.md` if needed for new features/changes
- Ensure documentation reflects current functionality

### 4. Pre-Commit Verification
- Run `pnpm test` — fix any failures immediately
- Run `pnpm start` — verify no runtime errors
- Only proceed when all tests pass

### 5. Commit and Push
- Update version in `package.json` to next sequential version
- Identify the most significant change for this release
- `git add . && git commit -m "VERSION_NUMBER - EMOJI SHORT_TITLE"` (version + emoji + main feature)
- `git push origin main` — triggers GitHub Actions auto-publish

### 6. Verify npm Publication
- Poll npm registry for 5 minutes:
  ```bash
  for i in $(seq 1 30); do sleep 10; v=$(npm view free-coding-models version 2>/dev/null); echo "Attempt $i: npm version = $v"; if [ "$v" = "NEW_VERSION" ]; then echo "✅ published!"; break; fi; done
  ```

### 7. Final Verification
- `npm install -g free-coding-models@NEW_VERSION`
- `free-coding-models --help` — verify binary works globally
- Only confirm release when global npm-installed version functions correctly

**Critical:** Never skip versions — consolidate all changes into the next sequential version number.

---
> Source: [vava-nessa/free-coding-models](https://github.com/vava-nessa/free-coding-models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
