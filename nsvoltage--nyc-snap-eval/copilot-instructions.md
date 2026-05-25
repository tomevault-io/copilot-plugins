## nyc-snap-eval

> Operating manual for Claude Code on this repository. Read this first, every session.

# CLAUDE.md

Operating manual for Claude Code on this repository. Read this first, every session.

## What we are building

A statute-grounded evaluation suite for caseworker-facing SNAP benefits-navigation AI in New York, plus a narrow reference implementation. The artifact is meant to look like an Anthropic research release — published grader, validated judge, honest limitations, generous credit, clear scope. The audience is a hiring manager on Anthropic's Beneficial Deployments team.

The system the eval is anchored to — NYC's MyCity chatbot — was discontinued by the Mamdani administration on 2026-02-05. Category A is therefore a post-mortem regression set rather than a live A/B comparator; see [ADR-008](docs/decisions/ADR-008-post-obbba-post-mycity-amendments.md) §4. The eval's value is reinforced by the shutdown, not undercut by it — the new administration's decision is consistent with the project's thesis that a measurable bar should have been applied before deployment.

## The single most important thing

**This is an illustrative suite, not a benchmark.** Approximately seventy cases across seven categories, scoped to one program in one jurisdiction at one point in time. Every design choice should serve that scope. If a decision would push us toward "comprehensive coverage of all SNAP eligibility," push back — that's not what we're building. We are demonstrating a way of thinking about responsible evaluation, with a working implementation as proof the eval is well-formed.

## The four documents that govern this work

In order of precedence when guidance conflicts:

1. **`docs/conventions.md`** — code style, voice, file layout. Hard rules.
2. **`docs/project-plan.md`** — the v2 build plan, day-by-day. What ships in v1, what is forward work.
3. **`docs/decisions/`** — Architecture Decision Records. Why we chose Promptfoo over MLflow, why caseworker-facing over applicant-facing, etc. These are the answers to "but what about X."
4. **`docs/research/`** — the four upstream research reports. Reference material for design judgment calls. Cite them when relevant, don't repeat them.

If you find yourself about to deviate from any of these, stop and ask the human.

## How we work together

The human is the HITL — they make the calls on scope, voice, and trade-offs. Your job is to execute, surface decisions cleanly, and flag drift. Specifically:

- **Surface before deciding.** When you hit a fork — should this be a CSV or YAML? Should the judge be Sonnet or Opus? — ask. Don't pick the more impressive option without surfacing the choice.
- **Default to the simpler option.** Propel's entire infrastructure is 14 lines of YAML and a Google Sheet. We are not building Propel + complexity. We are building Propel + statute grounding + judge transparency. That's it.
- **Match the voice in `docs/conventions.md`.** No marketing language. No "we're excited to announce." No emoji. No superlatives about our own work. Read the voice section before writing any prose.
- **Cite specific files when relevant.** "Per ADR-003" or "per `docs/research/r3-synthetic-data.md` §4" is better than vague gestures at the plan.
- **Don't expand scope.** If a task is "author the MyCity replay cases," don't also refactor the eval harness. Surface the refactor as a separate suggestion.

## Hard rules

These do not bend. If a request would violate one, push back before acting.

1. **Every eval case must have a `citation` field that resolves to a public source.** No exceptions. If we can't cite it, we don't test it. See `docs/conventions.md` §2.
2. **The judge model is pinned and the meta-prompt is published.** Claude Opus 4.5 grades; the full prompt lives at `eval/graders/llm_judge_meta_prompt.md` and changes go through ADRs.
3. **Eligibility math is never computed by the LLM.** The reference implementation calls the NYC Benefits Screening API or a deterministic Python port. The LLM narrates the result. See ADR-002.
4. **No PII-realistic synthetic data.** Personas are composite archetypes drawn from cited demographic research. Ranges and categories, not single specific values that read like a real person. See `docs/research/r3-synthetic-data.md` §"Synthetic persona generation."
5. **The simulated baseline is labeled as a simulation, every time it's referenced.** It is not a reproduction of any deployed system. See ADR-004.
6. **Limitations are first-class.** Every claim has a paired limitation. The README, methodology PDF, and blog post each have explicit limitations sections enumerating at least five specific constraints. See `docs/conventions.md` §"Voice."
7. **Attribution is in the first 1,000 words of any public-facing document.** Propel, The Markup, GOV.UK GDS, Nava Labs, Code for America, Stanford RegLab. Named, with URLs. Not in a footer.

## Where to find things

- **The plan, in order:** `docs/project-plan.md` has the day-by-day sequencing.
- **Why we chose what we chose:** `docs/decisions/` — ADRs are short and dated, read the headers to find the relevant one.
- **The four research reports** (these are the upstream synthesis we built the plan from):
  - `docs/research/r1-nyc-poverty-and-benefits-infrastructure.md` — the demographic and infrastructure context for why this work matters
  - `docs/research/r2-eval-methodology-and-mycity-postmortem.md` — Anthropic publication template, public-sector eval gold standards, MyCity post-mortem
  - `docs/research/r3-propel-positioning-synthetic-data-voice.md` — Propel deep-dive, caseworker vs. applicant positioning, synthetic data methodology, Anthropic voice study
  - `docs/research/r4-corpus-access-verification.md` — which policy sources are accessible and how (partially superseded by r5)
  - `docs/research/r5-freshness-audit-2026-05-11.md` — point-in-time re-audit at Day-1 corpus pull; records the MyCity shutdown, OBBBA-era statute changes, and two factual corrections to ADR-005/r4
- **Code:** `eval/`, `demo/`, `scripts/` — see the README for the directory tour.

## Default tool choices

These are decided. Don't re-litigate without an ADR.

- **Eval harness:** Promptfoo (see ADR-001)
- **Authoring surface:** Google Sheet at https://docs.google.com/spreadsheets/d/1CV7O8qHMTYQVFHCVFkWIE_zeDsFIyUDovCDLCHPmAcc/edit (anyone-with-link can edit). Exported to per-category CSVs via `scripts/export_sheet_to_csv.py`; the committed CSVs in `eval/cases/` are the canonical record that Promptfoo reads. The two-step flow trades Propel's one-source-of-truth pattern for reproducibility per `docs/conventions.md` §6 (CI runs offline, every release pins case content via `git log`).
- **System under test (primary):** Claude Sonnet 4.5 via Anthropic API
- **Comparison models:** GPT-4o, Gemini 2.5 Pro, plus the simulated baseline
- **Judge model:** Claude Opus 4.5, pinned, with published meta-prompt
- **RAG store:** pgvector on Supabase (see ADR-005)
- **Embeddings:** voyage-3
- **Eligibility tool:** NYC Benefits Screening API (`screeningapidocs.cityofnewyork.us`)
- **Demo framework:** Next.js 15 + Vercel AI SDK
- **Demo hosting:** Vercel
- **Methodology PDF:** Typst or LaTeX, whichever ships faster

## When to ask the human

Always ask before:
- Adding a new dependency
- Changing the eval taxonomy (the seven categories are settled per ADR-006)
- Modifying the judge meta-prompt
- Changing any hard rule above
- Writing public-facing prose for the first time in a session (let the human sanity-check voice before you draft three pages)
- Spending more than ~30 minutes on a single task without checkpointing

Don't ask before:
- Routine code work that follows existing patterns
- Running and reporting eval results
- Pulling additional policy corpus files from the sources already approved in ADR-005
- Drafting ADRs for decisions that need them (then surface for review)

## What "done" looks like for v1

The repo ships when:
- ~70 cases authored, every one with a citation field that resolves
- SME review completed on a sampled subset; inter-rater agreement reported
- Reference implementation runs end-to-end on a deployed Vercel URL
- Simulated baseline runs against the same eval; deltas reported with confidence intervals
- Methodology PDF written, applying the tone checklist in `docs/conventions.md`
- README applies the tone checklist
- Blog post drafted
- Public GitHub repo is published
- A specific named contact for SME review has signed off on at least the MyCity replay set and the factual recall set

We are not waiting for perfection. Propel shipped 25 cases. We are shipping ~70 with stronger validation discipline. Both are illustrative.

## What v1 does not ship

These are explicitly forward work, framed as such in the methodology PDF:
- Applicant-facing rubrics
- Multilingual generation at L1 (we ship multilingual consistency tests in English, Spanish, and one Asian language; native-language generation rubrics are v2)
- Live caseworker user study
- Cross-program transfer (Medicaid, TANF, WIC, LIHEAP)
- HRA Policy Directive / Policy Bulletin grounding (the corpus isn't publicly canonical)

Do not let scope creep pull these into v1.

---
> Source: [NSvoltage/nyc-snap-eval](https://github.com/NSvoltage/nyc-snap-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
