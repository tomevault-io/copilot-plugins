## research-process-builder

> Factory that produces validated web research processes via self-annealing loops. Takes a research goal, generates search patterns, tests against real companies, scores accuracy, and iterates until 90%+ reliability. Output: portable `.md` process files any agent (Claude, Clay/Claygent, GPT) can follow.

# research-process-builder

Factory that produces validated web research processes via self-annealing loops. Takes a research goal, generates search patterns, tests against real companies, scores accuracy, and iterates until 90%+ reliability. Output: portable `.md` process files any agent (Claude, Clay/Claygent, GPT) can follow.

## What It Produces

Two things:

1. **Research process files** (`processes/find-*/process.md`) — step-by-step search instructions with exact queries, extract specs, stop-if conditions, and kill lists. 20+ processes built (find-profiles, find-competitors, find-funding, find-series-a-daily, etc.).

2. **Scheduled monitors** (`monitors/`) — validated processes promoted to daily pipeline runs. Currently: `series-a-daily` (88% GT hit rate, $0.01/run via SerperDev).

## Install

No package manager file exists. Deps are Python stdlib + external APIs. Install manually:

```bash
pip install openai supabase requests
```

Required env vars (in `.env` at repo root):

```
OPENAI_API_KEY=
SERPER_API_KEY=
SUPABASE_URL=
SUPABASE_KEY=
```

Domain classifier also reads `DOMAIN_CLASSIFIER_KEY` if present (optional, falls back to conservative rejection).

**Env loading:** Global hook blocks direct `.env` reads. Scripts load via `dotenv` internally.

## Pipelines & Monitors

- **Series A pipeline:** `py scripts/series_a_pipeline.py` (supports `--dry-run`, `--stage N`, `--tbs qdr:w`, `--skip-enrich`, `--date`)
- **GT validation:** `py scripts/gt_validation.py --sample 10 --days 14` (add `--apply` to promote)
- **Domain classifier:** `py scripts/domain_classifier.py --domain example.com`
- **Prompt scoring:** `py prompts/[name]/score.py --prompt prompts/[name]/candidates/vNNN.json`
- **Monitors:** Orphan git branches mounted as worktrees. See `MONITORS.md` for setup.

## Building a New Research Process

Invoke the SKILL.md methodology (6 phases: define goal → generate 15-20 patterns → test across 3 company size tiers → score quality×consistency → iterate to 90%+ → assemble process file). Output goes to `processes/[name]/process.md`. Update `processes/[name]/STATUS.md` when done.

To graduate a validated process to a scheduled monitor, follow `MONITORS.md`.

## Key Conventions

- **`py` not `python`** throughout all scripts and docs (Windows default).
- **Single braces in GPT templates** — extraction prompt uses `.replace("{items}", payload)`, not f-strings. JSON examples inside the template need literal braces. Do not convert to f-string or `.format()`.
- **1-based local batch idx** in `extract_companies_batch` — model returns local idx, code maps `batch[local_idx-1]["idx"]` back to global. Keep this contract if re-annealing.
- **Supabase workspace 3 = production.** Always verify you're hitting the right table before any write.
- Ground truth files: `ground-truth/[company].json` — schema in `ground-truth/schema.json`. Baselines in `baselines/`.

---
> Source: [LeadGrowGTM/research-process-builder](https://github.com/LeadGrowGTM/research-process-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
