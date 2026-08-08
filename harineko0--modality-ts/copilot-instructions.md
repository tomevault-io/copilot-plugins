## modality-ts

> Avoid stopgap fixes and aim for fundamental solutions and future-proof improvements.

# Repository Guidelines

## Principles
Avoid stopgap fixes and aim for fundamental solutions and future-proof improvements.
Rather than overfitting to a specific library or framework, abstract it so that it can accommodate different ones. 
Never consider backward compatibility as this tool is experimental.

## Project Structure & Module Organization

`modality-ts` is a TypeScript ESM package for model-checking React state-transition behavior. Core library code lives in `src/`, organized into four top-level areas: `core/` for the public `modality-ts/kernel` API, `extract/` for TS/TSX extraction and source plugins, `check/` for the model checker, and `cli/` for CLI commands, codegen, replay harness, runtime assertions, and ambient shims. Tests mirror those areas under `test/`, with Vitest files named `*.test.ts`. Example React apps and their `app.props.ts` models live in `examples/`. Architecture and feature specs are in `docs/` and `docs/_specs/`; keep these aligned with behavior changes. `docs/` is the user-facing documentation, while `docs/_specs/` is the internal specification documentation.

## Build, Test, and Development Commands

Use `pnpm install` to install dependencies. Important checks:

- `pnpm typecheck` or `pnpm build`: run `tsc -b`; `build` emits `dist/`.
- `pnpm fix`: run `biome lint --write . && biome format --write .`.
- `pnpm test`: run the full Vitest suite.
- `pnpm architecture`: validate dependency rules with dependency-cruiser.
- `pnpm ci:examples`: run example-app integration checks.
- `pnpm phase7`: run differential checks for checker semantics, model generation, or TLA+ parity changes.
- `pnpm clean`: remove generated build output before a fresh build.

## Coding Style & Naming Conventions

Use strict TypeScript with NodeNext ESM imports. Keep modules small and colocated with their domain folder. Prefer exported interfaces and discriminated unions for IR and model shapes. Follow the existing style: two-space indentation, double quotes, semicolons, `camelCase` functions/variables, `PascalCase` types/interfaces, and kebab-case folders for source adapters such as `use-state`.

## Testing Guidelines

Vitest is configured for `test/**/*.test.ts` and `src/**/*.test.ts`. Add focused tests next to the affected subsystem under `test/<area>/`. Update or add coverage when changing extraction, checking, replay, reporting, source adapters, or CLI behavior. For architecture-sensitive imports, run `pnpm architecture`; for semantics-sensitive work, also run `pnpm phase7`. Also run `pnpm fix` to lint and format code.

## Commit & Pull Request Guidelines

Recent history uses short imperative subjects, often with an initial capital, for example `Fix CLI positional parsing` or `Restructure into root src and test directories`. Keep commits focused. Pull requests should describe user-visible behavior, list validation commands run, link related issues, and include screenshots only for example-app UI changes. Do not commit generated artifacts, local `.env` files, npm tokens, or `dist/`. Place worktree directories under `../modality-ts-worktrees`.

## Agent-Specific Instructions

When running shell commands in this repository, prefix commands with `rtk` where practical, for example `rtk pnpm test` or `rtk git diff`. Use raw commands only when debugging command filtering itself.

---
> Source: [Harineko0/modality-ts](https://github.com/Harineko0/modality-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
