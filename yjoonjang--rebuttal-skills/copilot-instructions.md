## rebuttal-skills

> This repo ships two skills for agents that can run shell commands.

# rebuttal-skills — agent guide

This repo ships two skills for agents that can run shell commands.

- **`rebuttaldraft`** — turn a paper plus its reviews into per-reviewer, paste-ready rebuttal drafts,
  running the experiments the rebuttal needs in the author's own workspace.
- **`reviewsearch`** — hybrid semantic search over **205,988 ML/NLP/CV peer reviews**
  (ICLR / ICML / NeurIPS / COLM) with their **author rebuttals**, via a hosted API. No local index or
  server needed.

**Install them properly rather than working from this file.** The skills are discovered natively once
they are on the skills path, in every workspace, which is what you want: the drafting workflow has to run
where the author's paper, reviews and code actually live, not inside this repo.

- Codex: [`.codex/INSTALL.md`](.codex/INSTALL.md) (clone, then symlink `skills/` into `~/.agents/skills/`)
- Claude Code: `/plugin marketplace add yjoonjang/rebuttal-skills` then `/plugin install rebuttal`

Every path inside a skill is **relative to that skill's own directory**, so the skills work in any host
that discovers them. `skills/rebuttaldraft/SKILL.md` is the drafting entry point and reads its stage
references from `references/`; it calls the search script in the sibling skill at
`../reviewsearch/scripts/search_reviews.py`.

## Searching directly

If you just want the corpus and are already working inside this repo, run the script yourself
(Python 3, standard library only, no dependencies):

```bash
python skills/reviewsearch/scripts/search_reviews.py "<query>" [--top-k N] [--accepted-only] [--year-min YYYY] [--year-max YYYY]
```

Example:

```bash
python skills/reviewsearch/scripts/search_reviews.py "reviews asking for a statistical significance test" --top-k 5
```

- Queries are **English** and work best as **concern-style natural language**, e.g.
  "missing ablation on learning rate", "reviewers doubting reproducibility".
- The script prints JSON: `{ count, results: [...] }`. Each result has `venue`, `year`, `title`,
  `decision`, `summary`, `concern` (reviewer weaknesses + questions — the main search target),
  `rebuttal` (the author's response; every result has one), and `score`.
- Filters map to venue years **2023–2026** and accept/reject decisions.
- Override the backend with the `RS_API_URL` env var. The free demo may **cold-start** (~1 min) after
  being idle — if a call times out, wait ~1 minute and retry once.

When you present results, cite the venue/year and note the data source (see Attribution).

## Two guarantees the drafting skill must keep

- **Confidentiality.** The paper and the review text never leave the machine. Only the short, abstract
  search queries reach the API. Never put paper content or a reviewer's wording into a query.
- **No fabrication.** Never invent an experiment result, a number, or a precedent citation. An
  experiment that was not run is answered honestly. With no compute at all, follow the dry-run rules in
  `skills/rebuttaldraft/references/stage3-experiments.md`: quarantine the output to `rebuttal_DRYRUN/`,
  never to `rebuttal/`.

**One consent gate:** Stage 3 presents a single consolidated plan table, and nothing runs until the user
approves it.

## Attribution

Data: peer reviews from **OpenReview** (ICLR / ICML / NeurIPS / COLM), licensed **CC-BY-4.0**.

---
> Source: [yjoonjang/rebuttal-skills](https://github.com/yjoonjang/rebuttal-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
