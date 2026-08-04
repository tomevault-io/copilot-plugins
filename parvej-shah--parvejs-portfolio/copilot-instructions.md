## parvejs-portfolio

> Behavioral guidelines for any agent (Claude or otherwise) working in this repo.

# CLAUDE.md — Operating Contract

Behavioral guidelines for any agent (Claude or otherwise) working in this repo.
This is the **first file to read**. It governs *how* to work here; `.agents/invariants.md`
governs *what must never break*; `.agents/memory.md` tracks continuity across sessions;
`.agents/features/*/purpose.md` + `rules.md` govern individual features/pipelines.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State your assumptions explicitly. If uncertain, ask.
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

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: every changed line should trace directly to the user's request.

**This is the guard against the most dangerous failure mode: a change that is locally
correct but globally harmful** ("this looks cleaner, I'll refactor it" — applied outside
the scope anyone asked for). Valid principle, wrong scope, and it erodes the intent baked
into surrounding code. When in doubt about scope, stop and ask rather than expand it.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require
constant clarification.

---

## ⭐ ENGINEERING STANDARDS — apply to EVERY task

1. **Follow MVC / layered architecture.** Keep concerns separated:
   - **Model** = Prisma models + `lib/validators` (Zod) + `lib/data` (data access). No HTTP here.
   - **Controller** = `app/api/**/route.ts` handlers — thin: parse → validate → call service/data → respond. No business logic inline.
   - **View** = React components / pages. No direct DB or fetch logic inside JSX; call services/hooks.
   - **Service layer** (`lib/services/*`) holds business logic shared by controllers. Routes stay skinny.
2. **Use design patterns where they fit** (and name them in code comments):
   Repository (data access), Singleton (Prisma/R2 clients), Factory/Builder for complex
   objects, Strategy for interchangeable behavior, Adapter around 3rd-party SDKs (R2 S3 SDK/NextAuth).
   Do not over-engineer — pattern must earn its place.
3. **Avoid code smells.** No duplication (DRY), no God functions/files, no magic numbers/strings
   (use constants/enums), no deep nesting (early returns), small single-responsibility functions,
   meaningful names, no dead code, no `any` (use inferred Zod types from `lib/types.ts`).
4. **Clean code hygiene:** consistent formatting, small commits with clear messages,
   handle errors explicitly (never swallow), validate all input at the boundary (Zod),
   never leak secrets/service keys to the client, guard every admin route.
5. **Reuse before you build.** Use existing `cn()` (`lib/utils.js`), shadcn `ui/*`,
   `buttonVariants`, `Reveal`, the `@/*` alias, and `lib/validators`/`lib/types` — do not
   duplicate types or re-declare shapes.
6. **TypeScript for all new code** (`.ts`/`.tsx`). Existing `.jsx` stays. `allowJs: true`.
7. **Respect file ownership** when working alongside another agent (see the relevant task
   list, e.g. `TASKS.md`, for the current split) to avoid merge conflicts.

---

## Anti-Refactoring Guards

- Never replace docs with aliases/summaries — extend them.
- Never delete a doc, invariant, or code block without explicit instruction.
- Never auto-deduplicate across features "for cleanliness" — duplication across
  feature boundaries is often intentional isolation, not waste.
- If a structural improvement would reduce usability or discoverability of these docs,
  **stop and ask** instead of applying it.

## Doc Hygiene (why this file structure exists)

Markdown project docs are structured context state, not disposable notes. Uncontrolled
growth (dumping every session's log) destroys signal as fast as having no docs at all.
So:

- `.agents/memory.md` stays **short and current** — active decisions and open threads,
  not a chronological archive. Prune stale entries when they're resolved; don't just append.
- Per-feature `purpose.md` + `rules.md` are scoped to that feature only. Don't let global
  concerns leak into them, and don't duplicate `.agents/invariants.md` inside them.
- `.agents/invariants.md` holds only things that must **never** be violated regardless of
  task framing. Keep it short — if everything is an invariant, nothing is.

## Read Order for Any Session

1. This file (`CLAUDE.md`).
2. `.agents/invariants.md` — hard constraints.
3. `.agents/memory.md` — current state, open decisions.
4. The relevant `.agents/features/<feature>/purpose.md` + `rules.md` for the area you're touching.
5. The active task doc (e.g. `TASKS.md`) for what to actually do.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due
to overcomplication, and clarifying questions come before implementation rather than after
mistakes.

---
> Source: [parvej-shah/parvejs-portfolio](https://github.com/parvej-shah/parvejs-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
