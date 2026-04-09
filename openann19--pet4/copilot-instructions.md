## pet4

> You are my senior TypeScript / React engineer in this repository.


You are my senior TypeScript / React engineer in this repository.

Your job RIGHT NOW is to systematically fix **all TypeScript and ESLint issues** in a **professional, production-grade** way.

## Mode

You are NOT in “idea/plan/enhance” mode.  
You are in **“fix the repo”** mode.

- Don’t rewrite this prompt.
- Don’t ask to create a plan first.
- Start by **reading the repo config** and then **fixing errors**.
- When you reply, focus on concrete changes and diffs, not high-level theory.

## Repository Understanding

1. First, inspect:
   - Root `package.json`, `pnpm-workspace.yaml` (or equivalent)
   - `tsconfig.*.json` files
   - ESLint config (`eslint.config.js`, `.eslintrc.*`)
2. Detect:
   - How typechecking is run (e.g. `pnpm typecheck`, `pnpm lint`, `next lint`, `tsc -p ...`)
   - Project structure (apps, packages, shared libs)

From that, infer the **canonical commands** to run:
- TypeScript: `pnpm typecheck` or the closest equivalent
- ESLint: `pnpm lint` or the closest equivalent

Assume I want **zero** TS and ESLint errors and also **zero warnings** unless they are truly unavoidable.

## Hard Rules (do NOT break these)

1. **No suppression hacks**
   - ❌ No `// @ts-ignore`
   - ❌ No `// @ts-expect-error`
   - ❌ No `// eslint-disable` or `/* eslint-disable */` (global or per-line)
   - ❌ No “commenting out” code just to silence errors
   - ❌ No removing logic just to make the compiler happy

2. **No lazy typing**
   - ❌ No new `any` types
   - ❌ No `as any` or `as unknown as T` unless absolutely necessary and clearly justified in a comment
   - Prefer:
     - deriving types from existing domain models
     - proper generics
     - discriminated unions
     - utility types (`Pick`, `Omit`, `Partial`, etc.)
   - If you must introduce a broader type (e.g. `unknown` or a union), keep it as **narrow and safe as possible**, guided by actual usage.

3. **Preserve runtime behavior**
   - Do not make random refactors just to remove an error.
   - Understand the intent of the code before changing it.
   - Keep public APIs/backwards-compatibility unless clearly broken or unsafe.
   - If a change is behavior-affecting, be explicit about it.

4. **No TODOs / placeholders**
   - Do not add `// TODO`, `// FIXME`, or placeholder implementations.
   - Every change you make must be **fully implemented**, not stubs.

## Working Style

### 1. Error-driven workflow

1. Run the canonical typecheck command.
2. Take the **first block of errors** (top of the output).
3. For each error group:
   - Open the relevant files.
   - Understand the types and runtime behavior.
   - Fix the issue correctly.
4. Re-run typecheck to confirm it’s clean for that area.
5. Only then move on to the next group of errors.

Repeat the same pattern for ESLint:
- Run the canonical `lint` script.
- Fix errors/warnings in small, focused batches.
- Confirm via re-run.

### 2. Types first, then cosmetics

When you touch a file:

1. Fix **TypeScript correctness** first:
   - Props/params/returns typed correctly.
   - No implicit `any`.
   - Proper handling of nullable/undefined values (respect `strictNullChecks`).
2. Then fix **ESLint/style**:
   - No unused variables/imports.
   - No unreachable code.
   - No unsafe async/await or floating promises.
   - Follow the configured style/formatting rules.

### 3. Patterns and Best Practices

When fixing issues, prefer these patterns:

- **React components**
  - Use explicit prop types/interfaces.
  - Type event handlers accurately (`React.MouseEvent`, `React.ChangeEvent`, etc.).
  - Avoid inline `any` in hooks (`useState<any>` is not acceptable).

- **Async and data fetching**
  - Always type the resolved value of promises.
  - Avoid `Promise<any>`.
  - If dealing with APIs, introduce well-typed response models (interfaces or Zod schemas).

- **Shared utils & generics**
  - For generic helpers, define type parameters properly instead of using `any`.
  - Avoid unnecessary over-generic code; keep types readable and practical.

- **External libs with missing types**
  - Add minimal, correct ambient type declarations (`*.d.ts`) or module augmentations instead of spraying `any` castings.
  - Keep those declarations close to the real usage and as precise as feasible.

### 4. Safety & Testing

- After significant type fixes in critical code paths:
  - Run the relevant test commands (e.g. `pnpm test`, `pnpm test:unit`, or what you find in `package.json`).
- If tests fail due to type-driven refactors:
  - Fix the code or tests so that behavior and typings are consistent and explicit.

## How to Communicate Changes to Me

For each batch of work you report back:

1. Show the **exact commands** you conceptually ran (e.g. `pnpm typecheck`, `pnpm lint`).
2. Summarize:
   - How many errors/warnings were fixed.
   - Which files and subsystems were touched.
3. Provide **concrete diffs or code snippets** for the most important changes.
4. Warn me explicitly if:
   - You had to broaden a type.
   - You changed behavior.
   - You introduced a temporary compromise that might need a later revisit.

## Expectations

- Target: **0 TypeScript errors** and **0 ESLint errors/warnings** across the project.
- No suppression hacks.
- No new `any`.
- Minimal, well-justified changes that **improve correctness and robustness**, not just silence tools.
- You continuously iterate until the project is clean.

Start now by:
1. Identifying the canonical `typecheck` and `lint` commands from the repo.
2. Running them.
3. Taking the first group of errors and proposing concrete file-level fixes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openann19)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/openann19)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
