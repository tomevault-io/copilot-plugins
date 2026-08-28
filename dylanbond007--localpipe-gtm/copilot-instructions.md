## localpipe-gtm

> The user describes a lead list in plain English. You turn it into a job spec and run it.

# LocalPipe GTM Engine — operating protocol

The user describes a lead list in plain English. You turn it into a job spec and run it.
They should never have to write JSON, run curl, or think about batching.

> "Every med spa in the Phoenix metro with 4+ stars, owner names and emails"
> "Plumbers, HVAC and electricians across the 10 biggest Texas cities — owner email + direct phone"
> "All the dentists in these 40 zip codes, but only the ones without a website"
> "Enrich this CSV I exported from Apollo"

## The protocol

Follow this every time, in order. Never skip step 2 or step 5.

**1 — Write a job spec** to `jobs/<slug>.json` ([schema](docs/reference/job-spec.md)).
Infer sensible defaults. Don't interrogate the user for fields they didn't mention.

**2 — Plan it, and actually read the output.**

```bash
python3 engine/lp.py plan jobs/<slug>.json
```

Show them the plan block. If a category resolved with `! low confidence`, ask which they meant
before running — scraping the wrong Google category wastes the entire run and their credits.

**3 — Decide whether to just go.**

| MAX CREDITS | What you do |
|---|---|
| ≤ 1,000 | Launch it. Tell them you did. |
| > 1,000 or unbounded | Show the ceiling, get an explicit yes first. |

Credits are the user's money. A Find-ALL sweep across 25 cities with phones on is a five-figure ask.

**4 — Launch detached.** Returns instantly; the run outlives your shell timeout.

```bash
python3 engine/lp.py run jobs/<slug>.json --detach
```

**5 — Poll to completion. Do not end your turn until you see `[DONE]` or `[FAILED]`.**

```bash
python3 engine/lp.py status
```

Each call is short and safe. Repeat every 30–60s between other work and paste the latest status
line so it doesn't look like a hang. Jobs legitimately run 5–30+ minutes; still-queued after 20
minutes is normal (fair-share queue) and the log says so.

**6 — Deliver.** Report lead count, owner-name/email/phone hit rates, and the CSV path. Then:

```bash
python3 scripts/preview_csv.py runs/<run>/results.csv
```

Send it with SendUserFile. Say plainly which emails are safe to send to — `owner_email` is
verified, `business_email` is **not** on credit-based plans.

## Job spec

```jsonc
{
  "name": "phoenix-med-spas",
  "search": {
    "queries": ["med spa"],              // plain English; auto-resolved to Google categories
    "locations": ["Phoenix, AZ", "Scottsdale, AZ", "85018"],
    "limit": "all",                      // "all" = Find ALL mode, or an integer PER PAIR
    "filters": { "min_rating": 4.0, "has_website": true },
    "relevance_filter": "off",           // "moderate"/"strict" for broad terms like "consulting"
    "resolve_categories": true           // false = pass their words through verbatim
  },
  "dedupe": true,
  "enrich": {
    "enabled": true,
    "want_email": true,                  // 2 credits
    "want_phone": false,                 // 10 credits — only when they ask for phones
    "want_general_email": true,          // 1 credit; returns as `business_email`, UNVERIFIED
    "general_email_fallback_only": true, // only when no owner email was found
    "decision_maker_title": null,        // e.g. "Practice Manager"; null = the owner
    "require_website": false,            // true skips no-website leads (cheaper, higher yield)
    "max_leads": null                    // hard budget cap; the engine logs loudly if it truncates
  }
}
```

To enrich a list they already have, swap `search` for a source:

```jsonc
{ "name": "enrich-apollo-export", "source": { "csv": "./inbox/leads.csv" }, "enrich": { ... } }
```

Any CSV with a business-name column works. Headers are auto-mapped (`Business Name`, `Website`,
`Address`… or snake_case). Rows without a name are skipped and reported.

## Translating what they said

| They say | You set |
|---|---|
| "everything / all of them / full coverage" | `"limit": "all"` |
| "a sample / just test it / top 50" | integer `limit` + `enrich.max_leads` |
| "no website" (web-design prospecting) | `filters.has_website: false` |
| "well-reviewed / established" | `min_rating: 4.0`, `min_reviews: 25` |
| "direct phones / cell numbers" | `want_phone: true` — **call out the 10-credit cost** |
| "any email is fine" | `want_general_email: true`, fallback off |
| "the office manager, not the owner" | `decision_maker_title: "Office Manager"` |
| a metro area | expand to the actual suburb list — coverage comes from listing suburbs |

**`limit` is per keyword × location pair, not a total.** 5 keywords × 10 cities × 300 = 15,000 leads.

## What things cost

| Field | Credits | Notes |
|---|---|---|
| `want_email` (owner, verified) | **2** | The default. Almost always on. |
| `want_general_email` (`business_email`) | **1** | Unverified. Use as a fallback. |
| `want_phone` (owner direct) | **10** | The expensive one. Only when they ask. |
| Google Maps scraping | **free** | Census a market for nothing. |

These are the published rates from [localpipe.io](https://www.localpipe.io). Credits are spent
**only when contact data is successfully found**, so the plan block's `MAX CREDITS` is a ceiling,
not a bill. See [docs/reference/credits.md](docs/reference/credits.md).

## Hard rules

Violating these silently loses the user's data or money.

1. **Never loop cities or keywords.** `query` and `location` are arrays — 5 keywords × 25 locations
   in ONE call. The engine chunks into the minimum number of requests. Never `for city in ...`.
2. **Never call the API from a tool call.** Submit → poll → paginate → enrich → poll all live
   inside `lp.py`. If you're about to curl LocalPipe, extend the engine instead. (`curl` is
   denied in `.claude/settings.json` for exactly this reason.)
3. **Never add a timeout to a poll loop.** Exit only on terminal status. Jobs never expire server-side.
4. **Always retry 429.** A 429 means the record was *not* accepted — no job, no webhook, no trace
   in the dashboard. The engine throttles to 460/min and retries. Don't bypass it.
5. **Always paginate `/get-scrape-results`** until `has_more` is false.
6. **Enrich submissions must equal lead count.** The engine asserts this and logs `!! COUNT
   MISMATCH`. If you see one, run `resume` — never report the run as complete.
7. **202 is success**, not an error.
8. **`"not found"` is the API's literal negative.** The engine blanks it in the CSV. Never let that
   string reach a CRM.
9. **Field names are exact:** `business_email` (not `general_email`), `owner_email` (not
   `personal_email`), `owner_first_name` (not `first_name`).
10. **Never print the API key**, never read `.env` into context, never paste a key into an issue.

## Commands

```bash
python3 engine/lp.py doctor                      # verify setup — zero credits
python3 engine/lp.py plan   jobs/x.json          # resolve, batch, estimate cost
python3 engine/lp.py run    jobs/x.json --detach # launch end-to-end
python3 engine/lp.py status [runs/<run>]         # newest run if omitted
python3 engine/lp.py resume runs/<run> --detach  # continue after any interruption
python3 engine/lp.py cancel runs/<run>           # stop in-flight; partials preserved
python3 engine/lp.py csv    runs/<run>           # re-emit CSV from what's done so far

python3 engine/categories.py resolve "med spas" "roofers"   # what will actually be searched
python3 engine/categories.py expand  "restaurant"           # every subcategory
python3 engine/categories.py grep    "roof"                 # raw substring search

python3 scripts/preview_csv.py runs/<run>/results.csv       # hit rates + send-readiness
```

## Layout

```
engine/lp.py            the engine — one process owns the whole chain
engine/categories.py    plain English -> Google's 4,285-category taxonomy
data/categories.csv     the taxonomy (ships with the repo)
jobs/examples/          starter job specs
runs/<name>-<stamp>/    run.log · state.json · leads.jsonl · enriched.jsonl · results.csv · DONE|FAILED
playbooks/              vertical GTM plays with ready-to-run specs
docs/                   tutorials, how-tos, reference
```

Runs are resumable: every phase writes to disk before advancing, so `resume` picks up exactly
where it stopped and re-submits nothing.

## When something breaks

| Symptom | Cause | Fix |
|---|---|---|
| `401 Invalid API key` | Key wrong, or deactivated | `python3 engine/lp.py doctor`, then regenerate at localpipe.io |
| `402` | Out of credits | Top up |
| `!! COUNT MISMATCH` | Submissions < leads | `resume` the run |
| `!! FATAL` then abort | Systemic failure (bad key) | Fix the cause, then `resume` — it resubmits only the gap |
| Job queued 20+ min | Fair-share queue, normal | Keep polling; jobs never expire |
| Zero leads | Over-filtering | Drop `min_rating`/`min_reviews`, or widen the category |
| Low email hit rate | Thin web presence in that vertical | Add `want_general_email` with fallback |

Full troubleshooting: [docs/how-to/troubleshooting.md](docs/how-to/troubleshooting.md)

---
> Source: [dylanbond007/localpipe-gtm](https://github.com/dylanbond007/localpipe-gtm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
