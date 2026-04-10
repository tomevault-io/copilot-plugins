## threadbox

> This repo’s core is a TypeScript DSL that transpiles directly into executable, type‑safe RxJS code. Agents will ultimately generate DSL programs and orchestrate sub‑agents, but today the DSL → transpilation → generated code pipeline is the mature foundation.

**Overview**

This repo’s core is a TypeScript DSL that transpiles directly into executable, type‑safe RxJS code. Agents will ultimately generate DSL programs and orchestrate sub‑agents, but today the DSL → transpilation → generated code pipeline is the mature foundation.

**Scope & Status**

- Core: DSL authoring, transpilation, and generated code checks are working
- Supporting: storage clients and Anthropic client are scaffolding for future integration
- Focus: reliability primitives (retry/backoff/recover) and composable, testable workflows

**Setup (per‑project tools)**

- Use `mise` to pin tools locally (no globals):
  - `bun` (runtime, test runner, bundler)
  - `just` (task runner)

```
# inside repo
mise use bun latest
mise use just latest
mise install
```

If using Homebrew instead: `brew install bun just`.

**Developer Workflow**

- Author check: `just dsl-check-author examples/risk_pipe.ts`
- Transpile: `just dsl-transpile examples/risk_pipe.ts ./.tmp/risk_pipe.gen.ts`
- Generated check: `just dsl-check-generated ./.tmp/risk_pipe.gen.ts`
- Tests: `just test` (set `POC_AGENT_DSL=1` to run only PoC tests)
- UUID benchmarks: `UUID_BENCHMARK_TESTS=1 just bench-id` (intensive performance tests)
- UUID randomness analysis: `UUID_RANDOMNESS_TESTS=1 just analyze-id-randomness` (intensive crypto tests)

**Debugging with Bun + VS Code**

Bun exposes a WebKit‑Inspector compatible with VS Code and a web debugger at `debug.bun.sh`.

Prereqs
- Bun ≥ 1.x (`bun --version`)
- VS Code “Bun for Visual Studio Code” extension
- `"sourceMap": true` in `tsconfig.json`

Just commands
- All tests, wait for attach
  ```bash
  just debug-tests
  ```

- Single test file
  ```bash
  just debug-tests-file tests/append.azurite.test.ts
  ```

Both commands start `bun test` with `--inspect-wait=6499`, pausing until the debugger attaches.

VS Code
1. Use the "Attach: Bun tests (port 6499)" config to attach.
2. Set breakpoints in .ts files; run continues on attach.

Tip: If attach is flaky on your platform, run `bun test --inspect=6499` and follow the printed debug.bun.sh link.

Ports & alternatives
- Change the inspector port by editing the just recipes, e.g. `--inspect-wait=9230`.
- For “break on first line”, replace with `--inspect-brk=PORT`.

**Supporting Infrastructure (Optional)**

An Azure Append Blob client and Azurite setup exist to support future storage operations and end‑to‑end demos. They are not the primary focus of the DSL pipeline. If you need Azurite to run blob‑related tests locally, consult the test files for connection string usage (`AZURE_STORAGE_CONNECTION_STRING=UseDevelopmentStorage=true`).

**Intensive Tests (Opt-in)**

Some tests are computationally intensive and disabled by default:
- **UUID Benchmark Tests**: Performance validation (500K+ UUIDs/second claims). Enable with `UUID_BENCHMARK_TESTS=1`
- **UUID Randomness Tests**: Cryptographic randomness quality analysis. Enable with `UUID_RANDOMNESS_TESTS=1`
- **PoC Tests**: Core DSL pipeline tests only. Enable with `POC_AGENT_DSL=1`

These flags follow the same pattern as `AZURE_STORAGE_CONNECTION_STRING` - set the environment variable to enable specific test suites.

CRITICAL: There must be no scripts in `package.json`. `just` and the `justfile` are the single source of truth for tasks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simbo1905)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/simbo1905)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
