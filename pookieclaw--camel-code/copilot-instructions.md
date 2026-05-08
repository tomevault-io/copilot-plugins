## camel-code

> An independent OCaml terminal AI coding assistant. Binary: `camel`.

# Camel Code

An independent OCaml terminal AI coding assistant. Binary: `camel`.
Not affiliated with Anthropic.

## Build

```bash
eval $(opam env)
dune build        # build
dune test         # run tests
dune exec camel   # run the binary
```

## Project Structure

```
bin/          Entry point (main.ml)
lib/          Core library (camel_lib)
test/         Alcotest test suite
```

## Conventions

- **OCaml 5.2+** with Eio for concurrency
- **dune** build system, single opam package
- **Conventional commits**: `feat(scope): ...`, `fix(scope): ...`, `refactor(scope): ...`
- **Feature branches**: `phase-N/descriptive-name`, merge to `main` at phase completion
- **No Co-Authored-By** lines in commits
- **Module naming**: snake_case files, match module names (e.g., `tool_registry.ml` -> `Tool_registry`)
- **Tests**: alcotest, one test file per major module
- **Error handling**: Result types, no exceptions in library code
- **Formatting**: ocamlformat with default profile

## Architecture

Built with:
- **lwd** for reactive UI (replaces React reconciler + virtual DOM)
- **nottui + notty** for terminal rendering (replaces Ink)
- **Eio** for structured concurrency (replaces async/await + generators)
- **First-class modules** for tool/plugin system (replaces TypeScript interfaces + buildTool)
- **Algebraic data types** for message/state types (replaces discriminated unions)

## Phases

0. Repo setup ✓
1. API streaming + REPL ✓
2. Tool system + 6 core tools ✓
3. TUI (notty/nottui/lwd) ✓
4. Sessions, config, commands ✓
5. MCP, agents, hooks, skills ✓
6. Vim mode, keybindings, permissions ✓
7. OAuth, bridge, coordinator (partial)
8. Polish, remaining tools, tests (in progress)
   - Prompt cache stability ✓
   - Tooled subagents (Read/Grep/Glob) ✓
   - Provider failover ✓
   - Lazy MCP connections ✓
   - Pre/post query hooks ✓
   - Session enrichment (git repo/branch/label) ✓
   - Doctor --fix ✓
   - Daemon mode (Unix socket) ✓

---
> Source: [pookieclaw/camel-code](https://github.com/pookieclaw/camel-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
