## bunv

> - `zig build` to build `bun`, `bunx`, and `bunv`

# AGENTS.md

## Build commands

- `zig build` to build `bun`, `bunx`, and `bunv`
- Build and run individual binaries with no arguments:
   - `zig build bun`
   - `zig build bunx`
   - `zig build bunv`
- To pass arguments to the binary, build and run it:
   ```sh
   zig build
   ./zig-out/bin/bun --version
   ```

## Code style

- Zig standard formatting

## Feature Work

When starting a feature, first create a plan at `plans/<feature-name>.md`. Iterate through the plan with the human and get explicit approval before starting work.

## Creating PRs

This project squashes and merges PRs, the title being the only text in the commit. Since the project uses conventional commits, the title MUST be in the format `<type>(<scope>): <subject>`. Scope is optional.

Otherwise you can use any format for commits, it's just the PR title that should be conventional.

---
> Source: [aklinker1/bunv](https://github.com/aklinker1/bunv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
