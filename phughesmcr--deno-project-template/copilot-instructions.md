## deno-project-template

> This is a personal experimental project and is greenfield — be ambitious. The purpose is working, deployable software delivered accretively in the shortest time compatible with correctness, performance, reliability, and innovation. Process exists to serve that outcome; it must never become the product.

# Project Guidance

This is a personal experimental project and is greenfield — be ambitious. The purpose is working, deployable software delivered accretively in the shortest time compatible with correctness, performance, reliability, and innovation. Process exists to serve that outcome; it must never become the product.

## Tech stack

- [Deno](https://docs.deno.com/) 2.9.0+ — runtime, test runner, formatter, linter, type checker
- [@std (JSR)](https://jsr.io/@std) — assert, path, fs, encoding, front-matter, media-types, etc.
- [Deno compile](https://docs.deno.com/reference/cli/compile/) — binary compilation when shipping a CLI or app
- Prefer Web Platform APIs and Deno APIs available on the stated Deno baseline

Dependency order: Web/Deno/`@std` → existing project deps → a well-maintained package → hand-roll only with a clear reason. Check docs and types before assuming a library lacks a capability.

If this project grows into a web or desktop app, prefer [Deno Fresh](https://usefresh.dev/docs/introduction) and [Deno Desktop](https://docs.deno.com/runtime/desktop/) over inventing a framework.

## Code Style

- Strict, functional TypeScript. Prefer data-driven design: plain data + functions over class hierarchies.
- Top-level functions must be `function` declarations, not `const` arrows.
- Use modern APIs where relevant (`Promise.try`, `Disposable` / `AsyncDisposable`, etc.) — only APIs available on Deno 2.9.0+.
- Formatting and linting rules live in `deno.json`. Honor them; don't fight them.
- Tests live in `__test__/`, suffixed `.test.ts`. Run with `deno test`.
- Filenames are lowercase `snake_case` (e.g. `entity_manager.ts`). Barrels are `mod.ts`, never `index.ts`.
- Imports are explicit and named, with the `.ts` extension — e.g. `import { spawnThing } from "../play/spawn.ts"`. No star-imports (`import * as ...`); they defeat tree-shaking for `deno compile`.
- Function names describe what they do, not where they live — the directory already says where.
- Do not preserve backward compatibility for unreleased or pre-1.0 surfaces. Remove obsolete paths instead of adding compatibility layers, fallbacks, or migrations. Treat published `exports` in `deno.json` as a contract once released.
- Choose the simplest implementation that fully meets current requirements. No speculative abstractions, configuration, or indirection.
- Grow in layers: smallest end-to-end slice first, then add capability on a product that already works. Long-term architecture means durable choices *after* a working slice — never trade a working product for unfinished complexity.
- Keep components modular and concerns separated.
- Remove dead code immediately (yours or pre-existing). Always delete imports, variables, and functions your changes orphan.

### Delivery integrity

- **No process or ceremony for its own sake.** Certificates, ledgers, dashboards, meta-reports, and paperwork are not progress unless they are a hard gate for a named feature.
- **Feature-first.** Almost all open work must deliver runnable behavior an end user or consuming agent can exercise. Process/ops items must name the feature they gate; ungated process work does not get created.
- **Honesty is absolute.** Never fake a test, present a fixture or mock as live proof, weaken an assertion to make it pass, hard-code a success path, or close work that is not done.
- **Refusal is not delivery.** A correctly typed refusal beats a fabricated result, but implementing only the refusal path never closes a feature. Mark refusal-only states explicitly and leave a follow-up for the real capability.

# Coding Guidance

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify. This is a small-scale open source project, not enterprise software.

## 3. Surgical Changes

**Touch only what you must.**

When editing existing code:

- Don't "improve" adjacent working code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.

This governs *working* code only — dead code is not protected (see Code Style). The test: every changed line traces to the user's request, or is dead code being removed.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

- "Add validation" → write tests for invalid inputs, then make them pass
- "Fix the bug" → write a test that reproduces it, then make it pass
- "Refactor X" → ensure tests pass before and after

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Don't abstract until the duplication is real

Prefer plain functions over factories, builders, and class hierarchies. A function plus a plain data structure beats a class with one method. No interface, DI container, or "manager" layer until at least two concrete callers need it.

Write the direct version first. Introduce abstraction the moment a second real caller forces it — not in anticipation of one.

## 6. Encode intent in types, not comments

Use the type system as documentation. Name constants. Model states as discriminated unions so illegal states are unrepresentable. Use a schema library (e.g. Zod) where data crosses a trust boundary — config, user input, persisted files, external payloads.

**Don't**

```typescript
// 1 = success, 2 = failure
function handle(result: { kind: number; value?: string; error?: string }) {
  if (result.kind === 1) { /* value is set... probably */ }
}
```

**Do**

```typescript
type Result =
  | { kind: "ok"; value: string }
  | { kind: "err"; error: string };

function handle(result: Result) {
  switch (result.kind) {
    case "ok":
      return useValue(result.value);
    case "err":
      return report(result.error);
    default: {
      const _exhaustive: never = result;
      return _exhaustive;
    }
  }
}
```

Rule: no bare numeric/string codes where a union or named constant fits. No optional field that's "only set when kind is X" — split the type. Switches over unions must be exhaustive (`never` in `default`).

## 7. Keep hot paths allocation-light

When a loop is genuinely hot (tight parse/transform, streaming, per-frame work in an app), allocate buffers once and mutate in place. Cold paths — startup, per-request setup, tests — may allocate freely.

**Don't** (inside a hot loop)

```typescript
function step(items: Item[]) {
  return items.map((p) => ({ ...p, x: p.x + p.vx })); // allocates every iteration
}
```

**Do**

```typescript
function makeBuffer(n: number) {
  const x = new Float32Array(n);
  const vx = new Float32Array(n);
  function step() {
    for (let i = 0; i < n; i++) x[i] += vx[i];
  }
  return { x, vx, step };
}
```

## Agent operations

- Prefer `deno task ci` before finishing a change (format, lint, types, docs, tests).
- Use `deno check` / `deno task check:types` while iterating; use full CI before claiming done.
- Request only the Deno permissions the code needs; don't widen `--allow-*` casually.
- Add packages to `deno.json` `imports` deliberately; prefer `@std` and existing deps first.

---

## Checklist before finishing a change

- [ ] Every changed line traces to the request, or is dead code being removed.
- [ ] No subsystem prefixes on function names; imports are named with the `.ts` extension.
- [ ] No new abstraction layer without ≥2 real callers.
- [ ] No magic numbers/string codes where a union or named constant fits.
- [ ] No allocation inside a genuine hot loop — cold paths exempt.
- [ ] Filenames are `snake_case`; barrels are `mod.ts`.
- [ ] `deno task ci` passes.

---
> Source: [phughesmcr/deno-project-template](https://github.com/phughesmcr/deno-project-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
