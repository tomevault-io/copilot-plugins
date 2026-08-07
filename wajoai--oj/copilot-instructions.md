## oj

> This repository is built for agentic TypeScript work. Keep changes small, verify them, and leave the repo easier to inspect than you found it.

# Agent Instructions

This repository is built for agentic TypeScript work. Keep changes small, verify them, and leave the repo easier to inspect than you found it.

The goal is idiomatic TypeScript with excellent feedback, not Rust-style TypeScript. Prefer TypeScript platform conventions, clear domain types, and verification that gives agents precise errors.

## Required Before Handoff

Run:

```sh
npm run verify
```

If that is too slow while iterating, run the narrowest failing check first, then finish with `npm run verify`.

## Dependency Rules

- Prefer the platform, TypeScript, and existing dependencies before adding packages.
- New packages must install through npm with the checked-in `.npmrc` policy.
- Do not bypass `min-release-age`, `save-exact`, or `ignore-scripts` without documenting the reason in the PR.
- Commit `package-lock.json` after dependency changes.

## TypeScript Rules

- Treat `any`, unchecked indexing, and broad object shapes as design smells.
- Prefer discriminated unions for state machines and command results.
- Use `assertNever` for unreachable union states and `assertPresent` instead of non-null assertions.
- Make union handling exhaustive; do not rely on default switch branches to hide missing cases.
- Avoid truthy checks for strings, numbers, and nullable values; narrow explicitly.
- Do not use `@ts-ignore`, `@ts-nocheck`, `as any`, or non-null assertions. If a suppression is unavoidable, explain why and keep it local.
- Use `unknown` at boundaries, validate it, then narrow it.
- Add `expect-type` assertions when a public API shape matters.
- Keep runtime validation close to external input boundaries.
- Keep files concise. The default limit is 500 LOC per file, 30 LOC per function (60 in test files), or 20 KB; split files by purpose instead of raising `.repo-health.json`.
- Never loosen `.repo-health.json` to silence findings — that is flagged as `diff.contractLoosened`. If a finding is genuinely unfixable right now, add a scoped entry to `exceptions` with a rule, path, justification (20+ characters), a `pr` referencing the pull request that de-risks the debt, and an `expires` date. No PR, no exception: the exception is invalid until the de-risking PR reference is present.
- Do not copy-paste near-identical functions (same file or across files), duplicate type declarations, repeat magic literals, or dodge size limits with names like `index2.ts`, `order-utils.ts`, or `doWorkPart2`; verification flags all of these.
- Do not give exported functions positional boolean flags or inline options objects with more than 8 properties; use named domain types or split the function.
- When verification reports `core.module.rewriteRecommended`, rewrite the module instead of patching individual findings; follow the repair plan at the top of the output and fix structure first, then types, then effects, then tests.
- During long repair loops, save a `--format json` result and pass it back with `--baseline` so you can see resolved versus newly introduced findings.

## Pull Request Notes

Include:

- What changed.
- How it was verified.
- Any dependency or security tradeoffs.

---
> Source: [WajoAI/OJ](https://github.com/WajoAI/OJ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
