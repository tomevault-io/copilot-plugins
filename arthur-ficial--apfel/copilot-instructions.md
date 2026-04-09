## apfel

> apfel has ONE purpose with THREE delivery modes:

# apfel - Project Instructions

## The Golden Goal

apfel has ONE purpose with THREE delivery modes:

> **Expose Apple's on-device FoundationModels LLM as a usable, powerful UNIX tool
> and an OpenAI API-compatible server, with a working command-line chat.**

### The three modes, in priority order:

1. **UNIX tool** (`apfel "prompt"`, `echo "text" | apfel`, `apfel --stream`)
   - Pipe-friendly, composable, correct exit codes
   - Works with `jq`, `xargs`, shell scripts
   - `--json` output for machine consumption
   - Respects `NO_COLOR`, `--quiet`, stdin detection

2. **OpenAI-compatible HTTP server** (`apfel --serve`)
   - Drop-in replacement for `openai.OpenAI(base_url="http://localhost:11434/v1")`
   - `/v1/chat/completions` (streaming + non-streaming)
   - `/v1/models`, `/health`, tool calling, `response_format`
   - Honest 501s for unsupported features (embeddings, legacy completions)
   - CORS for browser clients

3. **Command-line chat** (`apfel --chat`)
   - Interactive multi-turn with context window protection
   - Typed error display, context rotation when approaching limit
   - System prompt support

The Debug GUI has been extracted to its own repo: [apfel-gui](https://github.com/Arthur-Ficial/apfel-gui)

### Non-negotiable principles:

- **100% on-device.** No cloud, no API keys, no network for inference. Ever.
- **Honest about limitations.** 4096 token context, no embeddings, no vision - say so clearly.
- **Clean code, clean logic.** No hacks. Proper error types. Real token counts.
- **Swift 6 strict concurrency.** No data races.
- **Usable security.** Secure defaults that don't get in the way.

### Documentation style:

- **Links in docs and README:** Always use the URL/path as the anchor text, not generic phrases like "full guide" or "click here". Example: `[docs/background-service.md](docs/background-service.md)` not `[full guide](docs/background-service.md)`.

## Architecture

```
CLI (single/stream/chat) ──┐
                           ├─→ Session.swift → FoundationModels (on-device)
HTTP Server (/v1/*) ───────┘   ContextManager → Transcript API
                                ContextStrategy → 5 trimming strategies
                                SchemaConverter → DynamicGenerationSchema
                                TokenCounter → real tokenCount (SDK 26.4)
```

- `ApfelCore` library: pure Swift, no FoundationModels dependency, unit-testable
- Main target: FoundationModels integration, Hummingbird HTTP server
- Tests: `swift run apfel-tests` (pure Swift runner, no XCTest needed)
- No Xcode required - builds with Command Line Tools only

## Current Status

- Version source of truth: `.version` (currently `0.9.0`)
- Tests: `203` unit + `174` integration (full suite ~90 seconds)
- Issues `#33` through `#45` addressed
- v0.9.0: The Unification Refactor
  - Shared `processPrompt()` eliminates 5 duplicated code blocks between `singlePrompt()` and `chat()`
  - Chat+MCP crash fixed (#43): session created without requiring user message at init
  - `--debug` flag works in all modes (CLI, chat, server) - debug output to stderr (#44)
  - Ctrl-C exits chat cleanly (SIGINT handled via C shim around libedit)
  - Missing `ApfelError` cases: `unsupportedGuide`, `decodingFailure` (#41)
  - Context rotation bug fixed: MCP tools re-injected after rotation
  - Summarizer uses `makeModel(permissive:)` instead of `SystemLanguageModel.default`
  - Dead code removed: `sseStopChunk()`, `buildSystemPrompt()`, `formatToolResult()`
  - Homebrew formula: ARM check moved from hard error to caveats warning (#45)
  - 35 new chat integration tests (startup, exit, MCP, debug, JSON, Ctrl-C, multi-turn)
  - Integration test conftest auto-starts servers

## Build & Test

```bash
make install                   # bump patch + build release + install to /usr/local/bin
make build                     # bump patch + build release
make release-minor             # bump minor (0.6.x -> 0.7.0) + build
make release-major             # bump major (0.x.y -> 1.0.0) + build
make version                   # print current version
swift build                    # debug build (uses "dev" version stub)
swift run apfel-tests          # run pure Swift unit tests
```

**Version is in `.version` file** (single source of truth). Every `make build`/`make install` auto-bumps the patch number, updates README badge, and generates `Sources/BuildInfo.swift`. **Never manually edit `.version`, `BuildInfo.swift`, or the README badge** - always use `make build` which updates all three atomically.

**Always use `make install` for testing changes** - `swift run` uses a debug build, and the installed binary at `/usr/local/bin/apfel` won't reflect your changes until you run `make install`.

Integration tests (requires server running):
```bash
python3 -m pytest Tests/integration/ -v    # release-binary integration tests
```

Regenerate `docs/EXAMPLES.md` (runs 53 prompts against the installed binary, captures real unedited output):
```bash
bash scripts/generate-examples.sh          # ~2 minutes, overwrites docs/EXAMPLES.md
```

## Key Files

| Area | Files |
|------|-------|
| Entry point | `Sources/main.swift` |
| CLI commands | `Sources/CLI.swift` |
| HTTP server | `Sources/Server.swift`, `Sources/Handlers.swift` |
| Session mgmt | `Sources/Session.swift`, `Sources/ContextManager.swift` |
| Context strategies | `Sources/Core/ContextStrategy.swift`, `Sources/Summarizer.swift` |
| Tool calling | `Sources/Core/ToolCallHandler.swift`, `Sources/SchemaConverter.swift` |
| Token counting | `Sources/TokenCounter.swift` |
| Error types | `Sources/Core/ApfelError.swift` |
| Retry logic | `Sources/Core/Retry.swift` (withRetry, isRetryableError), `Sources/Retry.swift` (AsyncSemaphore) |
| Models/types | `Sources/Models.swift`, `Sources/ToolModels.swift` |
| Build info | `Sources/BuildInfo.swift` (auto-generated by `make`) |
| Security | `Sources/Core/OriginValidator.swift`, `Sources/SecurityMiddleware.swift` |
| MCP client | `Sources/Core/MCPProtocol.swift`, `Sources/MCPClient.swift` |
| MCP calculator | `mcp/calculator/server.py` |
| Tests | `Tests/apfelTests/` (188 unit), `Tests/integration/` (139 integration) |
| Tickets | `open-tickets/` |
| Docs | `docs/` (brew-install, EXAMPLES, release, tool-calling-guide) |
| Scripts | `scripts/generate-examples.sh` (regenerates docs/EXAMPLES.md), `scripts/write-homebrew-formula.sh` |

## Handling GitHub Issues

When a new issue comes in, follow this process:

1. **Fetch** the full issue with `gh issue view <n> --repo Arthur-Ficial/apfel --json body,comments,title,author,labels`
2. **Vet** - is it a real bug, valid feature request, or noise?
   - Does it align with the golden goal and non-negotiable principles?
   - Can you reproduce it?
   - Check comments for additional context and links
3. **Fix** if valid:
   - Write tests first (TDD) for bugs
   - Keep changes minimal and KISS
   - `make install` + run all tests (`swift run apfel-tests` + `python3 -m pytest Tests/integration/ -v`)
4. **Release** if code changed - see "Publishing a Release" below
5. **Close** the issue with a friendly, short, truthful comment:
   - What was the problem
   - What was fixed (or why it was closed without a fix)
   - How to update (`brew upgrade apfel`)
6. **Landing page** (apfel.franzai.com) is a separate Cloudflare Pages project, not in this repo

## Publishing a Release

### Preferred: GitHub Actions (fully automated)

The **Publish Release** workflow handles everything end-to-end:

1. Go to **Actions** tab in `Arthur-Ficial/apfel`
2. Run **Publish Release**, choose `patch`, `minor`, or `major`
3. The workflow will:
   - Bump `.version` via `make build` / `make release-minor` / `make release-major`
   - Build the release binary on `macos-26`
   - Run unit tests (`swift run apfel-tests`)
   - Commit `.version`, `README.md`, `Sources/BuildInfo.swift` and push to `main`
   - Create a git tag (`v<version>`) and push it
   - Package `apfel-<version>-arm64-macos.tar.gz` and publish a GitHub Release
   - Clone `Arthur-Ficial/homebrew-tap`, regenerate `Formula/apfel.rb` with the new URL + SHA256, commit and push
4. After the workflow completes, verify: `brew update && brew upgrade apfel && brew test apfel`

This is the preferred path for all releases. One click, fully tested, tap updated automatically.

### Fallback: manual local release

Use this only if the GitHub Actions runner is broken or you need to release from local changes that aren't pushed yet.

```bash
# 1. Build and install (auto-bumps patch version)
make install                           # bumps .version, builds release, installs to /usr/local/bin

# 2. Run ALL tests - no exceptions, no skips
swift run apfel-tests                  # unit tests (must be 188+)
# Start both servers for integration tests:
apfel --serve --port 11434 &
apfel --serve --port 11435 --mcp mcp/calculator/server.py &
sleep 3
python3 -m pytest Tests/integration/ -v   # integration tests (must be 139+, 0 skipped)
pkill -f "apfel --serve"

# 3. Package the release asset
make package-release-asset             # creates apfel-<version>-arm64-macos.tar.gz
make print-release-sha256              # SHA256 for homebrew formula

# 4. Commit, tag, push
git add .version README.md Sources/BuildInfo.swift <changed source files>
git commit -m "release: apfel v<version> - <summary>"
git tag -a "v<version>" -m "v<version>"
git push origin main
git push origin "v<version>"

# 5. Create GitHub Release
gh release create "v<version>" "apfel-<version>-arm64-macos.tar.gz" \
  --repo Arthur-Ficial/apfel \
  --title "v<version>" \
  --notes "Release notes here"

# 6. Update homebrew tap
git clone git@github.com:Arthur-Ficial/homebrew-tap.git /tmp/homebrew-tap
./scripts/write-homebrew-formula.sh \
  --version "<version>" \
  --sha256 "<sha256 from step 3>" \
  --output "/tmp/homebrew-tap/Formula/apfel.rb"
cd /tmp/homebrew-tap && git add Formula/apfel.rb && git commit -m "apfel v<version>" && git push

# 7. Verify brew install
brew update
brew reinstall Arthur-Ficial/tap/apfel
brew test Arthur-Ficial/tap/apfel
apfel --version                        # must show the new version
```

### Integration test rules

- **Never skip tests.** A skipped test is a critical error.
- Integration tests require two running servers: port 11434 (plain) and port 11435 (with MCP calculator).
- If servers aren't running, tests skip silently - this is NOT acceptable. Always start them.
- After releasing, run the full suite against the brew-installed binary as final verification.

### Post-release checklist

- [ ] Unit tests pass (188+)
- [ ] Integration tests pass (139+, 0 skipped)
- [ ] GitHub Release created with tarball
- [ ] Homebrew tap updated and `brew test` passes
- [ ] CLAUDE.md test counts and version updated
- [ ] File a ticket on `Arthur-Ficial/apfel-web` if the landing page shows test counts

## CI / GitHub Actions

- **`macos-26` runner** has Xcode-bundled SDKs (not standalone CLT). The workflow selects the latest available Xcode via `xcode-select` before building.
- apfel requires **SDK 26.4+** for FoundationModels token-counting APIs (`tokenCount`, `contextSize`). If the runner's highest Xcode is older, the build will fail.
- **`HOMEBREW_TAP_PUSH_TOKEN`** secret must exist on `Arthur-Ficial/apfel` - fine-grained token with Contents R/W on `Arthur-Ficial/homebrew-tap`.
- The **Publish Release** workflow (`.github/workflows/publish-release.yml`) is the single source of truth for the release pipeline. It handles version bump, build, test, tag, GitHub Release, and homebrew tap update in one run.
- The **CI** workflow (`.github/workflows/ci.yml`) runs on PRs and pushes for build + test validation.
- Release docs: `docs/release.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Arthur-Ficial)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Arthur-Ficial)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
