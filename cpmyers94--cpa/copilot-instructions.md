## cpa

> **This app exists to show what will and should happen to _each paycheck_.**

# Product north-star — read before building any feature

**This app exists to show what will and should happen to _each paycheck_.**
Every dollar the user tracks has to resolve to a **per-paycheck amount** — the
question a screen answers is always "what does *this* paycheck do?"

Concretely:

- **Everything needs a paycheck amount.** Income, bills, debts, expenses,
  goals, and savings must each express what they take from (or add to) a
  specific paycheck. A monthly or total-only figure is not finished — it must
  resolve to a per-paycheck contribution and be shown against the paycheck that
  covers it.
- **No orphan numbers.** A static balance or lump sum with no connection to a
  paycheck (e.g. "I have $250 saved") is incomplete on its own — pair it with
  the per-paycheck amount that funds or draws it down.
- When adding any new entity or field, ask first: *how does this show up on a
  paycheck?* If it can't, it doesn't belong in the plan.

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

---
> Source: [cpmyers94/cpa](https://github.com/cpmyers94/cpa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
