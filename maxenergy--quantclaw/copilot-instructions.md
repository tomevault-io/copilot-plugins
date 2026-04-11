## quantclaw

> `src/` and `include/quantclaw/` contain the C++17 core, split by subsystem (`core/`, `gateway/`, `tools/`, `web/`, `security/`, `plugins/`). `tests/` holds Google Test suites plus smoke checks like `smoke_test.sh`. `ui/` is the Vite control UI, `sidecar/` is the Node.js plugin sidecar, `adapters/` contains TypeScript chat adapters, `website/` builds the docs site, and `scripts/` provides build, release, and formatting helpers. Use `assets/` and `docs/` for bundled skills and notes.

# Repository Guidelines

## Project Structure & Module Organization
`src/` and `include/quantclaw/` contain the C++17 core, split by subsystem (`core/`, `gateway/`, `tools/`, `web/`, `security/`, `plugins/`). `tests/` holds Google Test suites plus smoke checks like `smoke_test.sh`. `ui/` is the Vite control UI, `sidecar/` is the Node.js plugin sidecar, `adapters/` contains TypeScript chat adapters, `website/` builds the docs site, and `scripts/` provides build, release, and formatting helpers. Use `assets/` and `docs/` for bundled skills and notes.

## Build, Test, and Development Commands
Use the root helpers when possible:

- `./scripts/build.sh --tests` configures and builds the C++ core and sidecar.
- `cmake -S . -B build -G Ninja -DBUILD_TESTS=ON && cmake --build build` is the direct equivalent.
- `cd build && ctest --output-on-failure` runs the full C++ test suite.
- `./build/quantclaw_tests --gtest_filter=AgentLoopTest.*` runs one Google Test suite.
- `bash tests/smoke_test.sh` exercises the gateway, REST API, and WebSocket RPC.
- `cd ui && npm install && npm run build && npm run test` builds and tests the dashboard.
- `cd sidecar && npm ci && npm run build && npm test` validates the plugin sidecar.

## Coding Style & Naming Conventions
C++ follows the Google style enforced by [`.clang-format`](/home/rogers/source/develop/QuantClaw/.clang-format): 2-space indentation, 80-column limit, attached braces, sorted includes, and `snake_case` file names like `session_manager.cpp`. Keep classes in `PascalCase`, functions and variables in `snake_case`, and mirror header/source pairs under `include/quantclaw/` and `src/`. TypeScript uses ES modules with explicit `.js` import suffixes where the package already does so; follow existing patterns such as `app-lifecycle.ts`, `config.browser.test.ts`, and `rpc-server.test.ts`.

## Testing Guidelines
Add or update tests for every behavior change. Name C++ tests `tests/test_<feature>.cpp`; use `*.test.ts`, `.browser.test.ts`, or `.node.test.ts` for UI and sidecar coverage. Run targeted tests locally before opening a PR, and run `tests/smoke_test.sh` for gateway, API, or concurrency changes. No fixed coverage threshold is defined, but untested feature work is not review-ready.

After any code change that affects runtime behavior, complete this verification loop before handoff: rebuild the project, restart `quantclaw`, confirm there is only one live `quantclaw` process, ask the user to send a Telegram message for validation, then inspect `~/.quantclaw/logs/` and the active process logs to confirm behavior and troubleshoot any failure.

## Commit & Pull Request Guidelines
Recent history mostly uses conventional prefixes such as `fix:`, `docs:`, `test:`, and `chore:`; prefer that format even though a few asset commits use short imperative subjects like `update logo`. Target `main`, link the related issue (`Fixes #123`), describe what changed and why, and call out breaking changes explicitly. PRs should pass local tests, add relevant test coverage, avoid new warnings, update docs for user-facing changes, and include screenshots when the UI changes.

## Configuration & Security Tips
Start from `config.example.json` and `scripts/env.example.txt`. Never commit keys, tokens, or machine-specific settings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxenergy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/maxenergy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
