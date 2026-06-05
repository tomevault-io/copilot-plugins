## repoprompt-classic

> - `RepoPrompt/`: macOS app source (Swift/SwiftUI), features, services, models, views.

# Repository Guidelines

## Project Structure & Module Organization
- `RepoPrompt/`: macOS app source (Swift/SwiftUI), features, services, models, views.
- `repoprompt-mcp/`: CLI client for MCP connection and discovery.
- `RepoPromptTests/` and `RepoPromptUITests/`: XCTest suites.

## Build, Test, and Development
- Target: macOS 14+ (Xcode 15+).
- New tests live in `RepoPromptTests/` with `*Tests.swift` naming.
- Use the standard `xcodebuild` CLI directly for Xcode builds and tests.

### Build and Test Commands

Discover project and scheme names when needed:

```bash
xcodebuild -list -project RepoPrompt.xcodeproj
```

Build and launch:

```bash
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug build
APP_PATH="$(xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug -showBuildSettings | awk -F ' = ' '/TARGET_BUILD_DIR/ { dir=$2 } /FULL_PRODUCT_NAME/ { name=$2 } END { print dir "/" name }')"
open "$APP_PATH"
```

Show build settings or clean build products:

```bash
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -showBuildSettings
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt clean
```

Testing rules:

- Never run the full test suite.
- Never run the full UI test target.
- Prefer `RepoPromptTests/` selectors by default.
- Use a targeted `RepoPromptUITests/` selector when UI or integration behavior is the most relevant validation path.
- Selectors should be target-prefixed, for example:
  - `RepoPromptTests/ApplyEditsCoreTests`
  - `RepoPromptTests/ApplyEditsCoreTests/testExactMatch`
  - `RepoPromptUITests/RepoPromptUITests/testAgentChatStressHarnessKeepsAutoFollowAndProducesGrouping`

Examples:

```bash
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt test -only-testing:RepoPromptTests/ApplyEditsCoreTests
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt test -only-testing:RepoPromptTests/ApplyEditsCoreTests/testExactMatch
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt test -only-testing:RepoPromptTests/PathMatcherTests -only-testing:RepoPromptTests/ApplyEditsCoreTests/testExactMatch
```

Notes:

- Prefer a single blocking `xcodebuild` command first.
- Use `-resultBundlePath <path>` when you need a result bundle for detailed failure analysis.
- Do not inspect PIDs or kill processes unless explicitly asked.

### Validation

Validate when a change can affect runtime behavior, buildability, packaging, MCP behavior, or CLI behavior.

- If you are already running a relevant `xcodebuild ... test ...`, do not run a separate build first.
- Otherwise run `xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug build`.
- Then run the smallest relevant test scope.
- Skip validation for docs, comments, prompt text, copy-only tweaks, or formatting-only changes.
- If changes affect the MCP server, CLI, or any feature that depends on the running app, validate with a local app launch plus the smallest useful CLI/MCP smoke check.

Use this flow when a feature needs live app interaction testing:

1. Build and launch the debug app:

```bash
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug build
APP_PATH="$(xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug -showBuildSettings | awk -F ' = ' '/TARGET_BUILD_DIR/ { dir=$2 } /FULL_PRODUCT_NAME/ { name=$2 } END { print dir "/" name }')"
open "$APP_PATH"
```

2. Connect with the debug CLI and run the smallest useful check for the feature under test:

```bash
rp-cli-debug -e 'windows'
rp-cli-debug -w 1 --call get_file_tree --json '{"type":"roots"}'
```

Then use `agent_run` to test agent functionality in the debug app, or use other targeted tools as needed to test non-agent functionality.

### Live Agent Mode / Claude Investigations

Before live Claude or Agent Mode smokes, enable debug-only diagnostics through the MCP `app_settings` surface of the debug app:

```bash
rp-cli-debug -w 1 --call app_settings --json '{"op":"list","group":"agent_mode","detailed":true}'
rp-cli-debug -w 1 --call app_settings --json '{"op":"set","key":"agent_mode.claude_raw_event_logging_enabled","value":true}'
rp-cli-debug -w 1 --call app_settings --json '{"op":"set","key":"agent_mode.claude_raw_event_log_file_path","value":"/tmp/repoprompt-claude-raw-events"}'
rp-cli-debug -w 1 --call app_settings --json '{"op":"set","key":"agent_mode.perf_diagnostics_enabled","value":true}'
```

These settings are intentionally DEBUG-only; if a key is unavailable, confirm you are connected to a current debug build before using lower-level UserDefaults fallbacks. Never hard-code investigation preferences in Swift source; keep them runtime-configurable and document exact keys/values in investigation notes.

### Symbolication Archives

For crash or performance investigations that need symbolication, check Xcode archives under:

```bash
~/Library/Developer/Xcode/Archives/<YYYY-MM-DD>/
```

Recent production archives may include paths like:

```bash
~/Library/Developer/Xcode/Archives/2026-05-13/RepoPrompt 2026-05-13, 10.55 AM.xcarchive
```

Use the archive's `dSYMs/Repo Prompt.app.dSYM` and verify the UUID matches the sample before running `atos`. If an investigation agent needs to locate archive folders or dSYMs outside the loaded RepoPrompt workspace, have it use bash commands such as `find`/`ls` rather than RepoPrompt file tools, which are scoped to loaded roots.

## Coding Style & Conventions
- Swift 5.x; targets macOS 14+; tab indentation (use tabs, not spaces).
- Types: `PascalCase`; methods/vars: `camelCase`; constants use `camelCase`.
- Match existing folder structure; keep changes minimal and focused.

## Testing Guidelines
- Add focused XCTest near related code; mirror file names (for example `PathMatcherTests.swift`).
- Prefer unit tests by default.
- Use targeted UI tests when they are the clearest or most reliable validation path.
- Keep tests fast, focused, independent, and deterministic.

## Commit Guidelines
- Commits: imperative mood, scoped and small (for example "Fix path matcher edge cases").
- Write a clear, descriptive commit message — descriptions matter. Explain the "why" (and the relevant "what") so someone scanning the log later understands the change without opening the diff.

### Parallel-agent safety (critical)
Other agents frequently work in this repo in parallel. You must not touch their work.

- **Never `git revert`, `git checkout --`, `git restore`, or `git reset` files you did not modify in this session.** Unrelated changes in the working tree or index may belong to another agent and reverting them destroys their work.
- **Never run bulk-stage commands** like `git add -A`, `git add .`, or `git commit -a`. These sweep up unrelated files.
- **Commit selectively.** Stage only the specific files you edited in this session by explicit path (`git add path/to/file1 path/to/file2`). Leave every other modified or untracked file alone, even if it looks stale, broken, or unrelated — it is not yours to clean up.
- **Before committing, run `git status` and confirm that only your intended files are staged.** If something you did not touch is staged, unstage it (`git restore --staged <path>`) before committing.
- If you think another agent's change is actively blocking your work, ask the user rather than reverting or "fixing" it yourself.

---
> Source: [repoprompt/repoprompt-classic](https://github.com/repoprompt/repoprompt-classic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
