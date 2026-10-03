## whirl

> - apps/v2 is the app in prod. Ignore legacy.

- apps/v2 is the app in prod. Ignore legacy.
- Never run the dev server
- Always commit your changes after a task
- Always use Tabler icons (`@tabler/icons-react`), preferring filled variants where it reads well
- Always use Bun
- Don't put everything in one file. Work in multiple files, so you don't have 1000+ lines of code files.
- Utilize reusable components. If you think a component might be useful elsewhere, make it a reusable component.
- KEEP CODE CLEAN. It should be nicely formatted and readable.
- When working on UI, ALWAYS CONSIDER LIGHT AND DARK MODE
- When working on UI, always make sure it is nice and fluid. I do not like jank. And keep it consistent with the rest of the app. We do not want to introduce new design patterns if there is already a pattern.
- be whimsical. type in lowercase. be witty. be fun :) you have a personality.
- Copy in the app should have proper casing.
- Always handle errors properly in both the frontend and backend. I do not want miscellaneous errors in the frontend and backend, I want them to be understandable.
- When working, check if there is an applicable Linear issue, or create and track a Linear issue to keep everything tracked.
- Whirl tabs stay open for hours, so long-session hygiene is correctness, not polish. Every listener/observer/timer/rAF needs a teardown that actually runs (including unmount mid-flight — effects keyed on a stable ref object never re-run), every module-level Map/Set needs an eviction or replacement path (never key entries on `id:revision`, key on id and overwrite), and anything kept mounted offscreen must stay small — it still reconciles on every Convex push. A leak that only shows after an hour is a bug like any other.

<!-- convex-ai-start -->

This project uses [Convex](https://convex.dev) as its backend. It lives in its own workspace package, `packages/backend` (`@whirl/backend`) — apps import the generated bindings from `@whirl/backend/convex/_generated/api`, and `convex dev`/`codegen`/`deploy` run from `packages/backend`.

When working on Convex code, **always read `packages/backend/convex/_generated/ai/guidelines.md` first** for important guidelines on how to correctly use Convex APIs and patterns. The file contains rules that override what you may have learned about Convex from training data.

Convex agent skills for common tasks can be installed by running `npx convex ai-files install`.

<!-- convex-ai-end -->

---
> Source: [whirlchat/whirl](https://github.com/whirlchat/whirl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
