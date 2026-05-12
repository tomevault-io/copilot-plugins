## modern-ai-developer-stack

> This file tells agentic coding tools (Cursor, Copilot, Windsurf, Claude, CLI agents) how to behave in this repository.

# AI Context & Agent Guidelines

This file tells agentic coding tools (Cursor, Copilot, Windsurf, Claude, CLI agents) how to behave in this repository.

The goal is to help engineers (and agents) maintain the "Modern AI Developer Stack" curated in `README.md` using predictable, scriptable workflows.

---

## 1. Repository Layout & Current State

- This repo is documentation-first, Awesome-style; the primary artifact is `README.md`.
- There is currently **no app or library at the root**:
  - No `package.json`, `pyproject.toml`, `Cargo.toml`, or similar.
  - No root build, lint, or test scripts.
- Root docs:
  - `README.md` – curated stack and main content.
  - `AGENTS.md` – this operational guide.
  - `LICENSE.md`, `CONTRIBUTING.md` – standard metadata (if present).
- If you add a subproject (Node, Python, Rust, etc.), create it in its own directory and document it:
  - In the subproject `README`.
  - In a short note in this file under the relevant language section.

Agents should assume that, unless a subproject defines a toolchain, only Markdown editing is needed.

---

## 2. Build / Lint / Test Commands

These conventions apply when you introduce code or tooling. Prefer simple commands that work in CI and are easy for agents to repeat.

### 2.1 Markdown Documentation (current root state)

The current repo is Markdown-only. Recommended commands **once a Node toolchain exists**:

```bash
npx markdownlint-cli "**/*.md"           # lint all markdown files
npx markdown-link-check README.md         # check links in the main doc
```

If you configure Node tooling, expose canonical scripts in that subproject:

```jsonc
{
  "scripts": {
    "lint:md": "markdownlint \"**/*.md\"",
    "test:links": "markdown-link-check README.md"
  }
}
```

### 2.2 Node / TypeScript Subprojects

When you add a Node/TypeScript subproject (in a subdirectory):

- Use `npm` or `pnpm` consistently inside that subproject.
- Add at least these scripts to its `package.json`:

```jsonc
{
  "scripts": {
    "build": "tsc -p .",
    "lint": "eslint .",
    "test": "vitest run" // or "jest"
  }
}
```

**Common commands (Vitest):**

```bash
npm test                                # run all tests (alias for vitest)
npx vitest run path/to/file.test.ts     # run a single test file
npx vitest run --testNamePattern "name" # run tests matching a name substring
```

**Common commands (Jest):**

```bash
npx jest                                # run all tests
npx jest path/to/file.test.ts           # run a single test file
npx jest -t "name substring"            # run tests matching a name substring
```

Agents should document any deviations (e.g., `mocha`, `playwright`) in the subproject `README` and, if important, in this file.

### 2.3 Python Subprojects

When you add Python-based tooling or examples:

- Use `uv` or `pip` + `venv` for dependency management.
- Default test runner: `pytest`.

**Common commands:**

```bash
pytest                                   # run all tests
pytest path/test_file.py::test_case      # run a single test function
ruff check .                             # lint (if ruff is used)
ruff format .                            # formatting (if ruff-format is used)
```

**Type checking (if configured):**

```bash
mypy .                                   # or: pyright
```

### 2.4 Other Languages (if introduced)

If you introduce additional languages, favor their standard workflows and document deviations here.

**Go:**

```bash
go test ./...                            # run all tests
go test ./... -run TestName              # run tests matching name
```

**Rust:**

```bash
cargo test                               # run all tests
cargo test test_name                     # run tests matching name
```

For any other language, follow its conventional `build` / `lint` / `test` commands and add a short section to this file.

---

## 3. Code Style Guidelines

These rules apply to any code added to this repository unless a subproject documents stricter rules.

### 3.1 Language & Types

- Prefer TypeScript over plain JavaScript for new Node-based code.
- Enable strict typing (`strict: true` in `tsconfig.json`) by default.
- Avoid `any` in TypeScript; use precise types or generics instead.
- For Python, use type hints and run static analysis (`mypy` or `pyright`) when feasible.
- Prefer explicit return types for exported functions and public APIs.

### 3.2 Imports & Modules

- Prefer ES modules (`import` / `export`) in JS/TS projects.
- Group imports with this order and a blank line between groups:
  1) Standard library.
  2) Third-party dependencies.
  3) Internal modules.
- Use path-mapped or short absolute imports instead of deep relative chains.
- Avoid wildcard imports except in test helpers or clearly scoped namespaces.
- Keep side-effect-only imports (`import "./polyfill"`) rare and clearly documented.

### 3.3 Formatting

- Default formatter: Prettier for JS/TS/JSON/Markdown when a Node toolchain exists.
- For Python, prefer `black` or `ruff format` for consistent formatting.
- Use LF line endings and avoid trailing whitespace.
- Target a line length of 100–120 characters.
- Do not rely on manual alignment that an autoformatter will undo.

### 3.4 Naming Conventions

- Variables and functions: `camelCase`.
- Classes, React components, and types/interfaces: `PascalCase`.
- Constants: `SCREAMING_SNAKE_CASE` for global or configuration constants only.
- Files:
  - Runtime modules and scripts: `kebab-case`.
  - React components: `PascalCase`.
  - Tests: mirror the file under test, e.g. `thing.ts` -> `thing.test.ts`.

### 3.5 Error Handling

- Fail fast in tooling and CLIs; do not silently ignore failures.
- Prefer language-standard error types, extending them only when necessary.
- Always provide actionable error messages with context (what failed and what to try next).
- For async JS/TS:
  - Use `try/catch` around the smallest block that can fail.
  - Avoid unhandled promise rejections; `await` or explicitly handle all promises.
- For CLI tools:
  - Exit with non-zero status codes on failure.
  - Print errors to stderr; keep messages concise but specific.

### 3.6 Testing Practices

- Prefer many small, focused tests over a few large integration suites.
- Keep tests deterministic; avoid live network or external API calls unless explicitly mocked.
- Name tests to describe behavior, not implementation details.
- For agent-focused tooling, include examples that can run in CI as part of the test suite.

---

## 4. Cursor & Copilot Rules

- As of this version, there is **no** `.cursor/rules/` directory, `.cursorrules` file, or `.github/copilot-instructions.md` in this repo.
- If you add Cursor or Copilot instruction files:
  - Ensure their constraints and workflows are consistent with this `AGENTS.md`.
  - Mirror any important rules here so all agents see a single source of truth.
  - Avoid IDE-specific rules that contradict the formatting, naming, or testing conventions above.

---

## 5. Agent Workflow Expectations

- Optimize for reproducible flows over ad hoc chat:
  - Prefer scripts, specs, and small utilities that other agents can invoke.
  - When adding commands, document them in both the code and the relevant README.
- Treat `README.md` as the product:
  - Keep entries focused on developer enablement, not marketing copy.
  - Align new tools with the existing taxonomy in the main README.
- When editing Markdown:
  - Maintain consistent heading levels and bullet styles.
  - Use concise, value-focused descriptions for tools and examples.
- When using git from an agent:
  - Avoid destructive commands (`reset --hard`, force pushes) unless the user explicitly requests them.
  - Do not modify unrelated files or revert user changes.

Agents should always ask:

> Does this change help developers build and maintain better software with AI, and is it easy for other agents to follow?

---

## 6. Skills & External Knowledge

- Prefer reusable skills for complex or repeatable workflows:
  - Use the Agent Skills format (folders with `SKILL.md`, scripts, and assets) when you capture a process that other agents should repeat.
  - Keep skills in version control so changes are reviewable and auditable.
- When designing a skill:
  - Make the skill single-responsibility (one clear capability per skill).
  - Include concrete examples and edge cases in the `SKILL.md` instructions.
  - Document required tools, APIs, and environment assumptions.
- When working with external knowledge tools (e.g., NotebookLM):
  - Treat them as grounded research backends; summarize findings back into this repo.
  - Do not rely on external tools as the only copy of critical instructions; mirror important outputs into `README.md`, `AGENTS.md`, or dedicated docs.

---
> Source: [eficode/modern-ai-developer-stack](https://github.com/eficode/modern-ai-developer-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
