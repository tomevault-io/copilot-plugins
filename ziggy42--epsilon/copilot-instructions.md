## epsilon

> - **Correctness, Performance & Security**: Absolute top priorities. When in doubt, consult the user on which to prioritize.

# Core Values

- **Correctness, Performance & Security**: Absolute top priorities. When in doubt, consult the user on which to prioritize.
- **Simplicity**: Secondary priority; must not be achieved at the expense of correctness, performance or security.

# General Rules

- **Dependencies**: Do not include any external dependencies. This is a **pure Go** project. **No CGo**.
- **Code Clarity**: Write self-explanatory code. Only add comments to explain genuinely non-obvious logic.
- **Scope**: Avoid creative additions unless explicitly requested.
- **Error Handling**: 
  - Never use `panic` unless explicitly instructed.
  - Do not ignore errors. Handle them or propagate them.
- **Naming Conventions**: Use self-explanatory variable names (e.g., use `name` instead of `n`).

# Git & Commits

- **Commit Messages**: Do not use conventional commit prefixes (e.g., `chore:`, `feat:`, `fix:`, `docs:`). Use plain, descriptive language.
- **Commit Authors**: Do not mark any commit as co-authored by agents. Commits should only include human authors.

# Project Specifics

- **Testing**:
  - Verify core logic changes against spec tests: `go test ./internal/spec_tests/...`
  - Ensure performance regressions are checked: `go test -bench . ./internal/benchmarks`
- **Performance**:
  - Minimize heap allocations in the hot path (VM instruction loop).
  - Prefer reusing buffers/slices where possible.

---
> Source: [ziggy42/epsilon](https://github.com/ziggy42/epsilon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
