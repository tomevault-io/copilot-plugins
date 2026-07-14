## ai-cost-cutter-skills

> AI spend has two honest levers: send less to the model (tokens), and send it to a cheaper model that provably still does the job (routing). Everything else is a vibe. Treat every cost decision as a claim that needs a check behind it, and say plainly which part is proven and which part must be measured on real traffic.

# Cost-cutter guidelines

AI spend has two honest levers: send less to the model (tokens), and send it to a cheaper model that provably still does the job (routing). Everything else is a vibe. Treat every cost decision as a claim that needs a check behind it, and say plainly which part is proven and which part must be measured on real traffic.

## Visibility before optimization

- Before optimizing anything, attribute spend by **tokens AND dollars** per model. Cheap models dominate volume, premium models dominate cost; never read a token ranking as a cost ranking, and never optimize the wrong line.
- The only honest savings number compares cost per **finished task** before and after, on the same workload. Cheaper per token is not cheaper per task if the cheap model needs extra loops.

## Routing

- Bulk work goes to a cheap default behind an OpenAI-compatible gateway; a premium model handles escalations behind a **stated gate** (task type, difficulty signal, or failed first attempt). An escalation rule that matches everything is the premium model with extra steps.
- When a cheap executor consults an expensive advisor: hard-cap the advisor calls, require a drift-check every N steps, and compute effective cost from the **actual** call count; a benchmark's "63% of the price" assumed a call rate your task doesn't have.
- Before any swap to a cheaper model (text, image, or video), list the cases the premium model dominates by a class (not a margin) and prove those still route premium. A blind swap saves money and quietly ships the failures.

## Tokens

- When an agent re-reads the same files every question, wire a queryable index (repo knowledge graph, RAG, repo map) so the shape becomes query-in, small-answer-out. Measure tokens on the same task before and after; keep it only if the drop is real and quality held.
- Reasoning effort is mostly output tokens, and often the biggest single lever on the bill. Default one notch below max; escalate per task by rule (failed attempt, tagged hard, irreversible step), never globally.

## Choosing and keeping models

- Choose models with a bake-off on the user's real prompts, with the selection criterion stated **before** running, sized to fit a free tier's caps. Include the incumbent; "keep what you have" is a valid result.
- Any workload that depends on one hosted model needs a pinned fallback that is open-weights, different from the primary, and carries a tested-on date plus the smoke prompts actually run. A backup you never ran is a hope.

## Free tiers

- Free tiers are for deciding and for patient one-time batches, never for live traffic. Size the batch to the rate limit and token budget before starting (items x tokens/item vs budget; items / (rpm x 60) hours ETA).
- Free models are the most likely to retain or train on inputs. Sensitive material goes to a cheap paid model; pennies, better policy.

## Always

- Cite prices and limits with their as-of date, from the current pricing page, never from memory.
- State which numbers a check proved and which the user must measure. A claim without a run behind it is marketing, not engineering.

---
> Source: [Neeeophytee/ai-cost-cutter-skills](https://github.com/Neeeophytee/ai-cost-cutter-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
