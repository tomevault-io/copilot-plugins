## epsilon

> - Brand-new language and codebase: prioritize fastest, simplest implementations.

# Repository Guidelines

## Project Philosophy
- Brand-new language and codebase: prioritize fastest, simplest implementations.
- No backwards-compatibility guarantees at this stage; prefer breaking changes over complexity.

## Project Structure & Module Organization
- Multi-host layout:
  - hosts/go/{go.mod, cmd/<tool>/, internal/<pkg>/, pkg/<lib>/}
  - hosts/odin/{cmd/<tool>/, pkg/<lib>/}
- Core language and shared libs:
  - core/{ast, ir, macros, schema, serde, transpiler}
  - shared/{runtime, stdlib}
- Tooling and tests:
  - tools/{repl, test_runner}
  - tests/{go, odin}
  - spec/fixtures/ (host-agnostic; do not duplicate)
  - docs/, assets/

## Language & Architecture Overview
- Lisp dialect for high performance using game engine techniques.
- Hosts: Go first; Odin later. Round-trippable AST (source ⇆ host AST).
- Execution: tree-walking interpreter; two-way transpiler to/from host AST.
- Syntax: between Fennel and Clojure. Core types: bool, nil, int64, float64, text (UTF‑8), binary, slice, map; keywords are interned strings. AST tags use short lowercase forms: i64, f64, txt, bin, slice, map, kwd, bool, nil.
- Mutability: use `put` for locals, slices, fields, and maps (setting a map key to `nil` deletes). `for` is end-exclusive, 0‑based.

## Build, Test, and Development Commands
- Use Taskfile (`Taskfile.yaml`) for workflows:
  - `task setup` — install dependencies (Python/Node)
  - `task lint` — run linters
  - `task fmt` — format code
  - `task test` — run tests
  - `task run` — run the app
  - `task dev` — dev/watch mode
- Discover tasks: `task --list`.

## Coding Style & Naming Conventions
- Indentation: Python 4 spaces; JS/TS/JSON/YAML 2 spaces. Max line length: 88 (Python), 100 (others).
- Naming: Python `snake_case` for functions/vars, `PascalCase` for classes. JS/TS `camelCase` for functions/vars, `PascalCase` for types/classes.
- Tools: Python `black` + `ruff`; JS/TS `prettier` + `eslint`. Run `task fmt` before opening a PR.

## Testing Guidelines
- Place tests under `tests/{go, odin}`; mirror host package structure and core where applicable.
- Fixtures live in `spec/fixtures/` and are host-agnostic; do not duplicate per host.
- Name tests `test_*.py` (Python) or `*.spec.ts` / `*.test.ts` (JS/TS) when applicable.
- Aim for ≥80% coverage on changed code. Prefer deterministic, fast tests.
- Run with `task test`.

## Commit & Pull Request Guidelines
- Do not try to create or push commits ever.
- Use Conventional Commit style in PR titles: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `ci:` (optional scope: `feat(api): ...`).
- Include a clear description, linked issues, and screenshots/CLI output where relevant. Note test coverage/impact.

## Security & Configuration
- Do not commit secrets. Provide `.env.example` and document required variables in `README.md`.
- Update `.gitignore` for `.env`, `.venv/`, `node_modules/`, and build artifacts. Pin dependencies when feasible.

## Agent-Specific Instructions
- Keep changes minimal and scoped to the request. Follow these conventions across new files.
- Prefer small, focused patches; update or add tests when changing behavior. Use `task` for setup, linting, testing, and running.
- Prefer using Serena tools wherever possible: lean on symbolic code operations and targeted search over ad‑hoc edits (e.g., `serena__find_symbol`, `serena__find_referencing_symbols`, `serena__insert_after_symbol`/`serena__replace_symbol_body`, `serena__search_for_pattern`, `serena__list_dir`). Keep plans up to date with `update_plan`, and use Serena thinking/memory tools judiciously to stay on track.
- Checklist hygiene: when a roadmap step is completed, update its checkbox in the relevant index (e.g., `spec/interop/go/INDEX.md`).

## HARD Policy: PEG and Schema Live in Macros (.e)

- Do not add or modify host builtins or special forms for PEG or Schema.
- All PEG and Schema logic must be implemented in the macro layer and `.e` files (e.g., `spec/macros/`, `spec/stdlib/`).
- The host may provide only generic primitives (e.g., txt/slice/map helpers) that are not PEG/Schema-specific.
- CI/Review should reject any change that introduces `peg/*` or `schema/*` host builtins.

---
> Source: [delaneyj/epsilon](https://github.com/delaneyj/epsilon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
