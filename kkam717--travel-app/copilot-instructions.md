## travel-app

> Graphify — refresh graph outputs when codebase changes touch graphed surfaces; read GRAPH_REPORT for architecture work


# Graphify (parallel to `ai/Context.md`)

Outputs live under **`graphify-out/`** (see **`ai/GRAPHIFY.md`**). This is **additive** to **`ai/Context.md`**, **`ai/APP_OVERVIEW_FOR_AI.md`**, and **`ai/UX_FLOWCHART.md`** — not a replacement.

## Important limitation (Flutter)

Upstream Graphify **does not classify `.dart` as code**, so **`lib/` is absent** from the structural graph. For **routes, shells, trip hub, and Dart architecture**, treat **`lib/router.dart`** + **`ai/Context.md`** as canonical.

## When to read

For **console, website, Supabase Edge TS, preview JSX, or native iOS/Android** structure and cross-file coupling, read **`ai/graphify/GRAPH_REPORT.md`** first (same content as **`graphify-out/GRAPH_REPORT.md`**, copied on every **`run_graphify.sh`**). Use **`ai/graphify/graph.json`** when you need the graph structure alongside other files under **`ai/`**.

If **`graphify-out/wiki/index.md`** exists, prefer that navigation over ad-hoc grepping.

## When to refresh (same session as edits)

After you **materially change** any of: **`console/`** (TS/TSX), **`website/`** front-end scripts, **`supabase/functions/`**, **`preview/`** reference trees, or **non-generated Swift/Kotlin** under **`ios/`** / **`android/`**, refresh from the repo root:

```bash
./scripts/run_graphify.sh
```

If outputs are **missing** or **stale** for a task, run the same command before relying on the graph.

### Completion cadence (keep the graph aligned with the repo)

Before considering a **code-changing** task **complete** (end of the same session / before wrap-up):

1. **If this session’s edits touched any graphed tree** above (or mixed **`lib/`** Dart with those trees), run **`./scripts/run_graphify.sh`** from the repo root so **`graphify-out/graph.json`** and **`graphify-out/GRAPH_REPORT.md`** match the current tree.
2. **If the only edits were Flutter Dart** under **`lib/`**, **`test/`**, or **`integration_test/`** with **no** changes to console, website JS/TS, Edge functions, preview, or native glue, **skip** the refresh — the code-only graph does not include `.dart` (see **`ai/GRAPHIFY.md`**).
3. **When in doubt** (e.g. large merge, scripted bulk edits, unclear paths), run the script anyway — it is quick and overwrites only **`graphify-out/`** outputs and the **`ai/graphify/`** mirror (not source).

**Do not** run Graphify as a substitute for updating **`ai/Context.md`** / **`ai/UX_FLOWCHART.md`** when **`lib/router.dart`** or user-visible app behavior changes (see **`.cursor/rules/update-context-md.mdc`**).

## Commits

When the refreshed graph is **useful to ship** (e.g. meaningful console or Edge changes), include **`graphify-out/GRAPH_REPORT.md`** and **`graphify-out/graph.json`** in the same change set when practical — and **`ai/graphify/`** (`GRAPH_REPORT.md`, `graph.json`, **`README.md`**) so the **`ai/`** bundle stays current for others. Never commit **`graphify-out/cache/`** (ignored).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kkam717) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
