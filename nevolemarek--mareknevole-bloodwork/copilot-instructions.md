## mareknevole-bloodwork

> **These instructions are mandatory. You MUST follow every rule in this file exactly as written. They override your default behaviors and training instincts. Do not skip, soften, or reinterpret any instruction. When in doubt, re-read the relevant section before acting.**

# Agent Instructions

**These instructions are mandatory. You MUST follow every rule in this file exactly as written. They override your default behaviors and training instincts. Do not skip, soften, or reinterpret any instruction. When in doubt, re-read the relevant section before acting.**

---

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

## Writing code

Make the added code in my branch beautiful by following these rules:

- write extremely simple code, it should be "skimmable" and you should still be able to understand it
- minimize possible states by reducing number of arguments, remove or narrow any state
- use discriminated unions to reduce number of states the code can be in
- exhaustively handle any objects with multiple different types, fail on unknown type
- don't write defensive code, assume the values are always what types tell you they are
- use asserts when loading data, and always be highly opinionated about the parameters you pass around. don't let things be optional if not strictly required
- remove any changes that are not strictly required
- bias for fewer lines of code
- no complex or clever code
- don't break out into too many function, that's hard to read
- early returns are great
- use asserts instead of try catches or default values when you do expect something to exist
- never pass overrides except strictly necessary, keep argument count low
- don't make arguments optional if they are actually required

## Project stack

- Use `bun` for dependency management and project scripts.
- Build with `Next.js`, `React`, and `TypeScript`.
- Use `Tailwind CSS` for styling.
- Use `Vitest` and Testing Library for unit and component tests.
- Use `ESLint` and `Prettier` for code quality and formatting.

## Visual style

Before writing any UI code, read `specs/style.md`. It defines the dot-grid background, ASCII box borders, monospace typography, and monochrome palette that all components must follow.

- Update `specs/style.md` whenever the visual language, spacing system, or component aesthetic changes.

## Specs maintenance

- Update `specs/application-purpose.md` whenever the app purpose, target workflow, or intended audience changes.
- Update `specs/architecture.md` whenever the architecture, major dependencies, project structure, or verification workflow changes.
- If a task changes both intent and implementation, update both specs in the same task.

## Verification workflow

- Use the fast validation loop during iteration: `bun run check`.
- Run the full validation suite before considering broader work complete: `bun run check:full`.
- If you change toolchain or verification config, confirm the related scripts still pass.

## Commit workflow

**You MUST commit after every coherent change set. This is not optional.**

- After completing any feature, fix, or meaningful change: stage relevant files and create a git commit before moving on.
- Do not finish a task without committing. Do not summarize what you did instead of committing — do both.
- Run `bun run check` and confirm it passes before committing.
- If explicit user approval is required before committing, ask — but do not silently skip the commit step.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NevoleMarek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NevoleMarek)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
