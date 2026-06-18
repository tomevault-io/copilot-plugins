## dbt-ai-data-diff

> >


# dbt AI Data Diff

Compare production vs development data for modified dbt models and
open an interactive HTML report in the browser.

## When to Use

Run data-diff **after `validate.sh` passes** (all steps green). The
models must already be built in the `tmp` schema.

Suggest running it when:

- The developer says "looks good", "validation passed", or "ready to
  commit" after modifying models
- You finish a model creation or modification task and validation
  succeeds
- The developer explicitly asks for a data diff or comparison
- A PR is about to be opened and no Datafold results are available yet

**Do NOT run it**:

- Before validation — models must be built in `tmp` first
- For models that only exist in production (nothing to compare)
- When the developer is still iterating on SQL and hasn't validated yet

## Run

```bash
.agents/skills/data-diff/data-diff.sh <model_selector>
```

### Selector examples

```bash
# Single model
.agents/skills/data-diff/data-diff.sh int_order_pricing

# Multiple models
.agents/skills/data-diff/data-diff.sh "int_order_pricing int_product_inventory"
```

The script prints progress to stderr and the output HTML file path to
stdout.  It automatically opens the page in the browser on macOS.

### What happens under the hood

| Step | What | Cost |
|------|------|------|
| 1. Identify models | Manifest diff (local) | Free |
| 2. Code diff | sqlglot AST parse (local) | Free |
| 3. Schema diff | `INFORMATION_SCHEMA` query | 1 fast query / model |
| 4. Extract primary keys | Manifest parsing (local) | Free |
| 5. Profile columns | BQ profiling query | 1 query / model / env |
| 6. Sample rows | `EXCEPT DISTINCT` query | 1 query / model |
| 7–8. Assemble + render | JSON → HTML injection | Free |

**Performance**: ~30–60 seconds for up to 5 models.

## Interpreting the Output

The HTML page has a summary section and per-model cards.

### Colour key

| Colour | Meaning |
|--------|---------|
| **Green** | Safe — no data change, or expected increase |
| **Amber** | Attention — unexpected metric shift (null spike, mean drift) |
| **Red** | Risk — large row count drop, columns removed, null spike |
| **"NEW" badge** | Model has no production counterpart — review the full profile |

### Per-model cards

Each card shows:

1. **Row count** — prod → dev with delta and % change
2. **Schema changes** — columns added / removed / type changed
3. **Code changes** — expression diffs (before → after) from sqlglot,
   CTE changes with a warning banner if upstream logic changed
4. **Column profile table** — side-by-side prod vs dev stats (distinct
   count, null %, min, max, mean).  Changed columns are highlighted.
5. **Sample rows** — expandable `<details>` sections with added,
   removed, and modified rows (capped at 10 each)

### Risk indicators

The summary section flags:

- Row count drops > 1%
- Null % increases > 5 percentage points
- Columns removed from schema
- CTE changes that indirectly affect all columns

## What to Tell the Developer

After the page opens, **summarise the key findings in chat**.  Focus on
what changed and whether it looks expected.

### Good summary format

> 3 models diffed.
>
> **int_order_pricing**: 340 fewer rows (−0.22%) — CTE
> `line_items` modified (WHERE filter added).
> `total_price_coalesced` expression changed: now falls back
> to 0.  Null % on `list_price` dropped 77.35% → 77.12%.
>
> **int_product_inventory**: 0 row change.  New column
> `reorder_threshold` (INT64, 41% null).
>
> **int_customer_segment**: NEW model — 143,892 rows, 5 columns, 2.56%
> null on `email`.
>
> See full diff: `.data-diff/data-diff-feature-update-pricing-20260314-143200.html`

### What to highlight

- Unexpected row count changes (especially drops)
- Columns where null % increased significantly
- Expression changes that could affect downstream consumers
- New columns with high null rates
- Any red risk indicators from the summary

## Prerequisites

Same as `validate.sh`:

- `.venv/bin/dbt` and `.venv/bin/python3` (with sqlglot) available
- `gcloud` authenticated to your GCP project
- `bq` and `jq` installed
- `target_prod/manifest.json` and `target/manifest.json` exist
- Modified models built in `tmp` schema

If any prerequisite is missing, the script exits with a clear error.

## Limitations

- **Modified models only** — does not profile downstream dependents
  (future iteration)
- **Requires validate.sh first** — models must be built in `tmp`
  before diffing
- **sqlglot parsing is best-effort** — complex Jinja models (heavy
  `{% for %}` / `{% if %}` blocks) fall back to compiled SQL or may
  produce partial results.  A warning is shown when this happens.
- **No incremental awareness** — profiles the full table, not just
  the incremental slice
- **Single environment** — compares `tmp` (dev) vs the model's
  production schema.  Does not support cross-environment comparisons.

## Integration with Other Skills

| Workflow step | Skill | When |
|---------------|-------|------|
| Build/modify a model | `create-*-model` | First |
| Validate it compiles, builds, passes tests | `validation` | Second |
| **Compare data against production** | **`data-diff`** | **Third** |
| Review for framework compliance | `review-model` | Fourth |

---
> Source: [affsantos/dbt-ai-data-diff](https://github.com/affsantos/dbt-ai-data-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
