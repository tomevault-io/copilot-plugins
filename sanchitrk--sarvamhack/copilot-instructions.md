## sarvamhack

> *PART OF ONGOING HACKATHON: OPTIMISE FOR HACKATON*

# Repository Guidelines

## Project Structure & Module Organization
*PART OF ONGOING HACKATHON: OPTIMISE FOR HACKATON*

This repository is a pnpm workspace for TypeScript packages. Root configuration lives in `package.json`, `pnpm-workspace.yaml`, `tsconfig.json`, and `vite.config.ts`.

- `packages/core/src/` contains the shared `@sarvamhack/core` library.
- `packages/core/tests/` contains its Vitest tests.
- Package build output is written to `dist/` and must not be committed.

## Task Completion Requirements

- `vp check` and `vp run typecheck` must pass before considering tasks completed, unless otherwise specified.
- Use `vp test` for the built-in Vite+ test command and `vp run test` when you specifically need the test package script.

## Code Style

- Optimise for Hackathon. No nonsense optimizations for maintainability.
- Always strive for concise, simple solutions.
- If a problem can be solved in a simpler way, propose it.
- No pre-mature optimizations, propose if required.
- Use single word filename whenever possible otherwise split into kabab-case(only if no options).

## Vendored Repositories

This project vendors external repositories under `.repos/` as read-only reference material for coding agents.

## General Preferences

- If asked to do too much work at once, stop and state that clearly.
- If computer use is helpful for completing or verifying work, use it.
- Avoid defensive future-facing design. This leads to pre-mature assumptions which are never true.

## Personal Notes & Document Spec

This project uses private design documents under `.notes/` as implementation reference for coding agents. When specified use it for planning, designing and brainstorming. When creating, mention in doc current date and git commit ID if available.

- Allow the design docs to be reviewed by subagents when see fit.
- Propose improvements when see fit.
- Optimize for Hackathon, we have limited time.

---
> Source: [sanchitrk/sarvamhack](https://github.com/sanchitrk/sarvamhack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
