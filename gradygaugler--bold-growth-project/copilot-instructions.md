## bold-growth-project

> This repo is a take-home for the **Product Manager, Agentic Growth** role at Bold.org. The deliverable is an **agentic growth workflow** (not a deck) that targets Bold's three public top-of-funnel surfaces — scholarship detail pages, scholarship index / by-* pages (SGPs), and the blog.

# AGENTS.md

## Purpose

This repo is a take-home for the **Product Manager, Agentic Growth** role at Bold.org. The deliverable is an **agentic growth workflow** (not a deck) that targets Bold's three public top-of-funnel surfaces — scholarship detail pages, scholarship index / by-* pages (SGPs), and the blog.

Per the prompt, the system must:

- Pick the highest-leverage problem solvable with an agentic system in ~3–4h.
- Run end-to-end, call a real LLM at least once, persist state where appropriate, and produce a useful artifact.
- Be shaped as one of: recurring loop, one-shot generator, or event-triggered — choice defended in the README.

Parts to deliver:

1. **Build** (3–4h) — runnable system + workflow artifact + sample output + ≤1-page README.
2. **Design two more** (1–2h) — ≤1 page each, at least one a different shape than Part 1.
3. **Avoiding fake wins** (≤30m) — instrumentation bugs, traffic-mix shifts, downstream quality regressions, hard-coded guardrails.

Senior-PM lens: data-grounded problem selection, taste about which agentic *shape* fits, and explicit trust-building before the system acts without human review.

## Reference Material

- `human/inputs/instructions.pdf` — full prompt + live page references.
- `human/inputs/data.xlsx` — sole dataset. Tabs:
  - `README` — definitions, funnel math, surface mapping.
  - `PAGE_TYPE_FUNNEL` — sessions → submit → verify → D7 activation, by surface.
  - `DEVICE_SPLIT` — desktop/mobile top-of-funnel + form diagnostics.
  - `TOP_PUBLIC_LANDING_PAGES` — per-URL GSC + funnel (46 pages).
  - `TECH_INDEX_SIGNALS` — sitemap, robots, llms.txt, canonical, redirects, structured data.
  - `EVENT_SANITY` — GA4 event counts + duplicate-firing diagnostics.
  - `EXPERIMENT_SUMMARY` — one live SGP by-year redesign A/B (flat result).

## Directory

- **`README.md`** — reviewer entry point.
- **`agent/`** — runnable agent + prompts.
- **`human/`** — prompt inputs, research notes, sample output, and Part 2/3 designs.
- **`mocks/`**, **`tests/`** — local fixtures and pytest coverage.
- Local runs write **`human/artifacts/`** and **`state/`** (mostly gitignored).

---
> Source: [GradyGaugler/bold-growth-project](https://github.com/GradyGaugler/bold-growth-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
