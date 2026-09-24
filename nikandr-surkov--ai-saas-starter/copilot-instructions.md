## ai-saas-starter

> Read AGENTS.md at the repo root — it is the full operating manual and wins over these lines.

Read AGENTS.md at the repo root — it is the full operating manual and wins over these lines.
Use pnpm only; done means `pnpm typecheck && pnpm lint && pnpm test` pass.
Never: edit applied migrations, mutate `credit_transactions`, touch credit tables outside `src/lib/credits/`, or weaken Stripe webhook signature checks.
Ask before adding dependencies, changing DB schema, or changing plans/prices.
UI changes follow DESIGN.md ("The Ledger"): 2px radius, no gradients/shadows, ledger rows over card grids, no emoji.

---
> Source: [nikandr-surkov/ai-saas-starter](https://github.com/nikandr-surkov/ai-saas-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
