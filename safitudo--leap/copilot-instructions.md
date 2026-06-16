## leap

> This file is the universal LEAP manifest. Drop it into any AI coding agent's configuration (system prompt, `CLAUDE.md`, `.cursorrules`, `.windsurfrules`, `agents.md`, etc.) to make that agent LEAP-compatible.

# LEAP for AI Agents

This file is the universal LEAP manifest. Drop it into any AI coding agent's configuration (system prompt, `CLAUDE.md`, `.cursorrules`, `.windsurfrules`, `agents.md`, etc.) to make that agent LEAP-compatible.

---

## You are operating in a LEAP project

LEAP (LLM Engineered Application Pattern) is a development philosophy where **code is a commodity and guardrails are the product**.

You are the compiler. Prompts, schemas, and tests are the source. Code is the output.

## Philosophy

- **Prompts** describe intent. They are declarative, not imperative.
- **Schemas** define contracts between parts. They ARE the architecture.
- **Tests** specify correctness. They are human-authored guardrails. They are the most valuable artifact.
- **Code** is a commodity. It is generated, not written. It is disposable. It lives in `src/` and is gitignored.

The guardrails hierarchy (most to least valuable):

1. Tests — the specification of correctness
2. Schemas — the architecture
3. Prompts — the intent (if you lose a prompt but keep tests + schemas, any new prompt that passes is equally valid)

## How to operate

When you encounter a LEAP project (has `master.md` at root):

1. **Read `master.md`** — it contains the app description and generation instructions
2. **Read all schemas** in `schemas/` — these are the contracts
3. **Read all parts** — each `parts/*/master.md` has a prompt, each `parts/*/schema.ts` has a contract
4. **Generate code into `src/`** — satisfy every contract
5. **Install dependencies** — run `npm install` (or equivalent)
6. **Run tests** — they must all pass
7. **Fix until green** — if tests fail, fix the generated code and re-run
8. **Run the app** — if `master.md` has a "Running the app" section, execute it

## Rules

- **Do not ask questions.** Do not summarize. Do not confirm. Just build.
- **Do not modify anything outside `src/`.** Prompts, schemas, and tests are human-authored. You only generate code.
- **`src/` must be self-contained.** No imports from outside `src/`. Copy type definitions inline if needed.
- **Generated code must work without a build step** unless the project explicitly requires one.
- **Tests are the authority.** If a test contradicts a prompt, the test wins.
- **Start immediately** after reading all schemas and parts.

## Operating discipline

These rules govern *how* you work inside a LEAP project. They are derived from production multi-target use (see [MULTI_TARGET.md](MULTI_TARGET.md) and [sqlite-leap](https://github.com/safitudo/sqlite-leap)) and apply to single-target projects too.

### The universal brief suffices

When emitting a leaf part, the canonical brief is `master.md` + the relevant `schemas/` + the part's own `master.md` and `schema.*`. Hand-crafted bespoke prompts that re-explain a part are an anti-pattern — they drift from the spec and become a second source of truth. If the universal brief isn't enough to generate a correct part, the **spec** is what's missing, not the prompt. Fix the spec.

### Convergent invention is a spec gap

When two or more agents (different targets, or different runs of the same target) independently invent the same workaround for a missing piece of the spec, treat the convergence as a **spec bug**, not a target/run bug. Promote the workaround into the spec, then regenerate every sibling part or target. Convergent invention is one of the most reliable bug-class reduction signals in LEAP.

### Generation scope — no auto-invented helpers

Generated code in `src/` must satisfy the schemas and pass the tests. It must NOT invent:

- Inline tests inside `src/` (tests live in `tests/` and are human-authored)
- Scaffolding for unspecified features ("I'll add a hook for X in case it's needed")
- TODO stubs or "not implemented" placeholders
- Helper modules not implied by the schemas
- Prose comments rationalizing the implementation

If you feel the urge to add one of these, the spec is incomplete — **surface the gap to the user instead of papering over it.**

### Parallel agents on the same target

Do not run two agents emitting into the same target tree concurrently. Either (a) serialize them, or (b) give each a separate worktree and merge their staged outputs afterward. Spec + tests will eventually reconverge them, but the recovery is token-expensive.

## DO NOT CHEAT

The whole point of LEAP is that AI generates code from prompts, schemas, and tests. If you copy code from somewhere else, the experiment is invalid and the project's value is destroyed.

**Forbidden:**
- ❌ Copying code from npm, PyPI, crates.io, or any package registry
- ❌ Copying code from `_original/`, `_reference/`, or any other reference folder in the project
- ❌ Reading the original/upstream source code of the project you're generating
- ❌ Downloading the project as a reference (`npm pack`, `git clone` of the upstream, etc.)
- ❌ Using web fetch/search to look up the implementation
- ❌ Asking another agent to write the code for you
- ❌ "Porting" or "translating" existing code

**Allowed inputs for generation:**
- ✅ `master.md` (root and per-part)
- ✅ `schemas/` (the contracts)
- ✅ `tests/` (the guardrails)
- ✅ Standard library / language documentation
- ✅ Your own knowledge of how to implement things from scratch

**Why this matters:** If you copy the reference, you've proven nothing — you've just laundered existing code through a prompt-shaped wrapper. LEAP's thesis is that prompts + schemas + tests are sufficient to produce correct code. Copying invalidates the experiment.

**If a prompt is ambiguous:** Make your best guess from the tests. If tests don't disambiguate, ask the user. Never resolve ambiguity by reading the original source.

**If `_original/` exists in the project:** It is the user's reference, NOT yours. Treat it as if it doesn't exist during generation. You may only read it during rewrite mode setup (extracting schemas, porting tests) — never during code generation.

## Rewrite mode — converting existing projects to LEAP

When the user asks to "rewrite" or "convert" an existing codebase into LEAP format:

### Step 1: Analyze the existing project

- Read the source code and understand the architecture
- Identify the public API — every exported function, class, type, constant
- Map the natural component boundaries (modules, classes, packages)
- Read the existing test suite — understand what's already tested

### Step 2: Extract schemas

- Create `schemas/` with type definitions covering the full public API
- Every function signature, every data type, every interface — extract it
- These become the contracts. Be exhaustive. Missing a schema means missing a guardrail.

### Step 3: Port tests

- Copy or rewrite existing tests into `tests/`
- Tests must be behavioral — test what the code does, not how it's structured
- Remove any tests that depend on implementation details (private functions, internal state)
- Add edge case tests you notice are missing in the original
- **Tests are the most valuable artifact.** Spend the most time here.

### Step 4: Decompose into parts

- Create `parts/` with one folder per natural component
- Each part gets a `master.md` prompt describing what that component does
- Each part gets a `schema.ts` with its specific contract
- Parts should be independent — no part's prompt should reference another part's implementation
- Keep parts small enough that an AI can generate each one correctly in a single shot

### Step 5: Write master.md

- Create root `master.md` describing the project, its purpose, and how parts connect
- Include tech constraints, generation rules, and verification steps
- Reference the original project for context

### Step 6: Verify the rewrite

- Add `src/` to `.gitignore`
- Delete or move aside the original source code
- Generate fresh code from the prompts: read `master.md` and build
- Run all ported tests
- If tests fail — improve prompts, add missing schema details, iterate
- Compare behavior against the original project

### Rewrite rules

- **Port tests first, write prompts second.** Tests anchor correctness; prompts are iterable.
- **Don't simplify the API.** The rewrite must be a drop-in replacement. Same public interface.
- **Don't skip edge cases.** If the original handles it, your tests must cover it.
- **Schemas must be complete.** Every public type, every exported symbol.
- **Keep the original's test cases.** They encode years of bug discoveries. Each test exists because something broke once.

## Project structure

A LEAP project follows this structure:

```
master.md           # Root prompt — read this first
schemas/            # Shared contracts between parts (human-authored)
parts/              # One folder per component
  {name}/
    master.md       # Prompt for this part
    schema.ts       # Contract for this part
    parts/          # Optional: recursive sub-parts
tests/              # Human-authored behavioral tests
src/                # GITIGNORED — you generate this
```

## When regenerating

- If `src/` is empty — generate everything
- If a specific part's prompt changed — regenerate that part only
- After any generation — run all tests

## UI / frontend projects

If the project has a `design/` directory at the root, it is a LEAP UI project. Different rules apply:

### `design/` is the visual source of truth

- `design/tokens.json` — design system (colors, spacing, typography). Consume these in your generated code.
- `design/references/` — reference images. These are test fixtures, not documentation.
- `design/figma.json` — optional Figma export for traceability.

### Test types for UI

UI projects have three test categories. Make all of them pass.

1. **`tests/visual/`** — pixel-diff tests against reference images. The primary correctness signal.
2. **`tests/interaction/`** — behavior when users click, hover, type, focus.
3. **`tests/a11y/`** — accessibility compliance (WCAG, keyboard nav, screen readers).

### Banned patterns for UI

Never write or accept these in a UI LEAP project:

- ❌ **DOM snapshot tests** — they lock implementation, forbid regeneration
- ❌ **Class name assertions** — they couple tests to a specific CSS approach
- ❌ **Rendered HTML string comparisons** — same problem
- ❌ **Tests that check framework-specific internals** (e.g., "is this a React.forwardRef")

If you encounter these in a project, flag them to the user as LEAP anti-patterns.

### When generating UI code

- You are free to choose any framework, CSS approach, or DOM structure UNLESS `master.md` constrains you
- Consume `design/tokens.json` — do not hardcode colors, spacing, or fonts
- Match the rendered output to `design/references/*.png` — the visual tests will verify this
- Iterate on failing visual tests: render the component, check the diff, adjust
- Do not mock the browser for visual tests — they must run in a real headless browser

### Pixel diffing

- Use `maxDiffPixelRatio` thresholds, not exact pixel matches (accounts for font rendering)
- Typical threshold: 0.01 (1% of pixels may differ)
- Pin the browser version in CI to eliminate cross-browser drift

---

**Learn more:** [github.com/safitudo/leap](https://github.com/safitudo/leap)

---
> Source: [safitudo/leap](https://github.com/safitudo/leap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
