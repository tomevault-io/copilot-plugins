## tasks

> > For Cursor/other AI tools: see `AGENTS.md`. For Gemini: see `GEMINI.md`.

# Claude Code Instructions

> For Cursor/other AI tools: see `AGENTS.md`. For Gemini: see `GEMINI.md`.

## Project

Stack: TypeScript · pnpm · SvelteKit · Vitest · Playwright · TailwindCSS · Drizzle · better-auth · Paraglide · MCP

## Critical Conventions (non-standard — always apply)

### Naming

- Variables / functions / params: `snake_case`
- Types / classes / interfaces / enums: `PascalCase`; enum members: `UPPER_CASE`
- Booleans: prefix `is_` / `has_` / `should_` / `can_` / `will_` / `did_`
- Constants: `UPPER_CASE` or `snake_case`

### Functions & exports

- Use `function` syntax, not arrow functions
- Multiple functions in a file: group into a namespace object `export { my_module }` (constants exempt)
- No `export default`

### Files

- Svelte: `PascalCase.svelte` / `PascalCase.svelte.ts` · TypeScript: `kebab-case.ts` · Route files: exception

### Quality limits

- Function complexity ≤4 · nesting ≤1 · function ≤25 lines · file ≤300 lines · params ≤3
- Magic numbers: extract all literals except `0`, `1`, `-1` to named `UPPER_CASE` constants
- No `any` · no unused vars · no floating promises · type assertions (`as`) are restricted
- All function params and return types must be explicitly typed
- Early return: single `return` under 100 chars → one-liner `if (x) return y`; otherwise block syntax

### Svelte

- `$state` reactive variables are reassignable
- Props interface name `Props` is allowed by ESLint
- DOM manipulation is restricted

### Content rules

- i18n: all user-visible strings (labels, buttons, toasts, validation errors, page titles) must use message keys — never hardcode. Add to all locale message files.
- Comments / test titles (`describe` / `it` / `test` / `expect` messages): English only
- No code duplication: extract to shared functions/modules immediately
- `/* @refactor-ignore */` at file top excludes a file from refactoring

## Code Change Rules

For every code modification, follow this order exactly:

1. **Refactor first** _(mandatory before lint or tests)_: apply high/medium-priority refactoring to all new/modified code — see `prompts/refactoring.md`. Do not proceed until no high/medium items remain.
2. **Tests**: add/update unit tests (Vitest) + E2E tests (Playwright) for all behavior changes — see `prompts/testing-guide.md`
   - **E2E cleanup / leaked data**: When fixing issues where E2E leaves database or UI artifacts, follow the **Regression fix workflow** in `prompts/testing-guide.md` (add a failing guard → fix → confirm green). Prefer stable selectors (`data-testid`) over locale-dependent strings for teardown.
3. **Lint**: run `pnpm run lint` then `pnpm run check`; fix all errors before reporting done
4. **Spell check**: `pnpm cspell:dot`; add legitimate project terms to `cspell.config.yaml`
5. **IDE feedback**: check IDE lint output — often more current than terminal
6. Never say "it should pass" without running commands. Never finish while errors exist.
7. Do not modify `eslint.config.js` unless explicitly asked; fix issues in application/test code instead.

## Completion gate (before you tell the user work is done)

Run the full verification set **in order**. **Do not** skip or reorder steps. **Do not** report completion if any step failed or was skipped without the user agreeing.

**STOP — Refactor before lint.** For any code change, you MUST complete refactoring (`prompts/refactoring.md`) **before** running lint or check. Do not run step 2 or later until refactoring is done. For a **refactor-only** request, follow `refactoring.md`'s own **convergence** (high/medium items until none remain).

**E2E:** The user runs `pnpm test` and shares the full output. Do **not** claim completion until the user confirms E2E passed or explicitly scopes it out.

1. **Refactor** — read and execute `prompts/refactoring.md` on all changed files. Converge until no high/medium items remain. **Do not proceed to step 2 until complete.**
2. `pnpm run lint`
3. `pnpm run check`
4. `pnpm cspell:dot`
5. `pnpm test:unit --run`
6. **IDE feedback**: zero **errors** on every file you changed (warnings only when documented as an allowed exception).
7. **E2E**: Ask the user to run `pnpm test` and share the output. Fix any failures, then ask again.

If you changed **only** docs or config that does not affect tests, still run lint + check + cspell; run unit tests when there is any chance of impact.

## Refactoring Rules

- When performing any refactoring, ALWAYS read and follow `prompts/refactoring.md` before starting.

## Git Rules

- **No commits** unless explicitly requested by the user
- For git operations: use `scripts/git-workflow.ts` via `pnpm git`

## Collaboration Workflow

- For issue-driven proposal/plan/execution/notification flow, follow `prompts/collaboration-workflow.md`

### Shorthand Commands

- `fullrun #<N>`: Post the agreed plan as an Issue #N comment → implement → `pnpm version:minor` → `pnpm git -y` → `pnpm git:followup` (full run from Step 3 onward in `prompts/collaboration-workflow.md`). Issue plan comments MUST be written in English. When running `pnpm git:followup`, compose an implementation summary in English and pass it via `--notify-message`. Format: `"<title>\n- <change1>\n- <change2>\n..."` (one bullet per meaningful change — what was added, changed, or fixed).
- `fullrun new` or `fullrun new "<title>"`: When no Issue exists yet (full run from Step 1 onward in `prompts/collaboration-workflow.md`). Steps: (1) Derive an English title from the conversation, or use the provided title. (2) Create Issue: `gh issue create --title "<title>" --body "<body>"` — body follows the minimum template in `prompts/collaboration-workflow.md`, filled from conversation context. Capture the new Issue number `<N>`. (3) Post the agreed plan as a comment in English: `gh issue comment <N> --body "<plan>"`. (4) Implement. (5) `pnpm version:minor`. (6) `pnpm git -y "<title> #<N>"`. (7) `pnpm git:followup "<title> #<N>" --notify-message "<title>\n- <change1>\n- <change2>\n..."` (one bullet per meaningful change).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshuafolkken) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
