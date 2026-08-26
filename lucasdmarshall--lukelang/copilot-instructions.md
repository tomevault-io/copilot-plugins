## lukelang

> This file is the canonical quick-start for AI/code agents working on LukeLang.

# AGENTS.md — LukeLang Agent Playbook

This file is the canonical quick-start for AI/code agents working on LukeLang.
If you have no LukeLang pretraining, read this file first, then execute tasks.

## 1) Project identity (do not drift)

- LukeLang is **Build-first**.
- `luke BUILD` is the language of record (native/WASM, no GC in shipped path).
- `luke SHOW --vm` is compatibility/convenience, not the product core.
- Backend + Live Graph are the current beachhead.

Primary docs:

- `docs/STRATEGY.md`
- `docs/BUILD_MODE.md`
- `docs/BACKEND_ROADMAP.md`
- `docs/LIVE_GRAPH.md`

## 2) Environment + build commands (canonical)

From repository root:

```bash
cd vm && make
```

Core commands (run from `vm/` unless noted):

```bash
./build/luke SHOW ../examples/build/hello.luke
./build/luke BUILD ../examples/build/hello.luke -o build/hello && ./build/hello
./build/luke BUILD ../examples/build/hello_wasm.luke -target wasm -o build/hello.wasm
./build/luke LSP
./build/luke DAP
./build/luke DEBUG ../examples/build/functions.luke --break 10 --batch
make test
```

Optional toolchain:

- WASM/browser output needs WASI SDK at `.tools/wasi-sdk` or `LUKE_WASI_SDK`.
- Browser/WASI smoke scripts use Node.js.
- Debugger checks require `gdb`.

## 3) Canonical examples (use these before inventing syntax)

Build/core:

- `examples/build/hello.luke`
- `examples/build/functions.luke`
- `examples/build/modules.luke`
- `examples/build/collections.luke`

Backend:

- `examples/build/backend_api.luke`
- `examples/build/backend_form_errors.luke`
- `examples/build/pg_api.luke`

Reactive/Live Graph:

- `examples/build/reactive_core.luke`
- `examples/build/live_graph_server.luke`
- `examples/build/live_graph_client.luke`
- `examples/build/live_graph_join_multi.luke`

Frontend (already shipped track):

- `examples/build/frontend_widgets.luke`
- `examples/build/frontend_wrap_forms.luke`
- `sample/landing.luke`

## 4) Compiler/diagnostic workflow

When changing parser, analyzer, compiler, LSP, DAP, or runtime-facing behavior:

1. Run a focused command first (single example).
2. Run targeted script checks.
3. Run `make test` before finalizing.

Useful checks:

```bash
bash scripts/debug_break_step.sh
bash scripts/debug_inspect.sh
bash scripts/dap_handshake.sh
bash scripts/lsp_providers.sh
bash scripts/fmt_roundtrip_all.sh
```

Diagnostics and formatter come from shared AST/build pipeline. Keep one truth path.

## 5) Tests and quality gates

Minimum expected before landing substantive runtime/tooling edits:

- `cd vm && make`
- `cd vm && make test-play` (if VM behavior touched)
- `cd vm && make test-build` (if Build/compiler/runtime touched)
- `cd vm && make test-lsp` (if editor/tooling touched)
- `cd vm && make test` for full gate

If full gate is too slow during iteration, document exactly which subset ran.

## 6) Code search patterns (fast navigation)

Use `rg` from repo root:

```bash
rg "THIS IS FUNCTION|REMEMBER|WHEN REACTIVE|WATCH|PUSH WATCH" vm/src/luke_parse.cpp
rg "luke LSP|LSP|hover|completion|semanticTokens" vm/src/lsp.cpp scripts/lsp_providers.sh
rg "DEBUG|DAP|gdb|inspect" vm/src/main.cpp vm/src/dap.cpp scripts/debug_*.sh
rg "httpServe|dbExecBind|dbQueryBind|pgQueryBind|pgExecBind" vm/runtime vm/stdlib examples/build
```

## 7) Module boundaries (keep architecture clean)

Compiler/runtime boundaries:

- `vm/src/luke_parse.cpp`: language parsing/statement recognition.
- `vm/src/luke_expr.cpp`: expression parser/eval helpers.
- `vm/src/build_c.cpp`: Build codegen and type/IR lowering.
- `vm/src/lsp.cpp`: LSP server over stdio.
- `vm/src/dap.cpp`: DAP server over stdio (gdb backend).
- `vm/src/main.cpp`: CLI command surface.
- `vm/runtime/*.h`: Build runtime helpers (arena, stdlib C bridge, net/db/auth/reactive).
- `vm/stdlib/*.luke`: Luke-level std modules.

Rule: do not reintroduce a parallel parser/toolchain path just for editor features.
LSP/FMT/diagnostics should stay aligned with the Build compiler truth.

Editor extension boundary:

- `tools/vscode/lukelang/` — VS Code client only (syntax, snippets, LSP/DAP launchers).
- Do not duplicate language logic in JS; call `vm/build/luke LSP` / `DAP`.

Package extension:

```bash
bash scripts/vscode_extension_package.sh
```

## 8) Backend-first publishing guidance

If the goal is backend adoption:

- Prioritize `std/server`, `std/sqlite`, `std/pg`, auth/session, migrations.
- Prefer correctness and diagnostics over syntax novelty.
- Preserve no-GC Build guarantees and predictable performance.
- Keep docs and examples runnable from CLI with minimal setup.

## 9) Documentation requirements for any feature PR

When adding/changing language behavior, include:

1. Doc update in `docs/` (or explicitly explain why not needed).
2. At least one canonical example under `examples/build/` (or tests proving behavior).
3. Regression test coverage in existing gates.
4. Clear command(s) to reproduce and verify.

## 10) Known non-goals while backend-first

- Do not shift core effort to mobile/game tracks.
- Do not add heavy ecosystem/distribution work unless requested.
- Do not prioritize stylistic syntax rewrites over backend + Live Graph milestones.

---
> Source: [lucasdmarshall/LukeLang](https://github.com/lucasdmarshall/LukeLang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
