## paper2beamer-skills

> This repository is a **reusable skills library** for reading, understanding, presenting, and synthesizing academic papers.

# AGENTS.md

This repository is a **reusable skills library** for reading, understanding, presenting, and synthesizing academic papers.
It contains two skills plus a copyable workspace template.
The full, detailed rules for each workflow live inside each skill's `SKILL.md` — this file only routes tasks and states the global principles.

```text
.
├── AGENTS.md                     ← you are here (routing + global principles)
├── README.md                     ← human-facing overview
├── skills/
│   ├── paper-beamer-deck/        ← single-paper deep-dive Beamer deck
│   └── research-gap-analysis/    ← multi-paper research-gap analysis
└── workspace-template/           ← copy this to start a new paper-reading workspace
```

## Highest-level principle

Do **not** mechanically copy paper text into slides or notes.
The goal is not PDF-to-slides or abstract-to-bullets conversion.
The goal is to read the paper(s) carefully, understand the technical content, reorganize ideas into a teachable structure, explain motivation/problem/method/evidence/results/limitations/implications, identify hidden assumptions and missing evaluations, and synthesize research opportunities when multiple papers are analyzed.

> The output should feel like a careful graduate student read the work, understood it, prepared a seminar talk or research memo, and thought critically about what it means.
> It should **never** feel like an LLM converted papers into shallow bullet points.

## Audience model

These skills are **field-agnostic**.
By default, assume a general graduate-level research audience in the paper's own field: technical, but they may not have read the paper.
Provide enough background to make the paper understandable without over-explaining basics that are standard in the field.

The audience/domain is a **per-workspace setting**.
If the workspace has a `domain.md` (see `workspace-template/`), use it to calibrate background depth, terminology, and examples to the actual research area.
Without it, stay field-neutral.

## Task routing

There are two task modes.
Pick the matching skill and follow its `SKILL.md` exactly.

| If the user asks to…                                                                 | Use this skill | Read |
|--------------------------------------------------------------------------------------|----------------|------|
| summarize, explain, teach, or **make slides for ONE paper**                          | **paper-beamer-deck** | [`skills/paper-beamer-deck/SKILL.md`](skills/paper-beamer-deck/SKILL.md) |
| analyze, compare, or **find research gaps across MULTIPLE papers**                    | **research-gap-analysis** (memo + a synthesis slide deck) | [`skills/research-gap-analysis/SKILL.md`](skills/research-gap-analysis/SKILL.md) |

**Single-paper examples:** "Summarize this paper and make slides." · "Create a Beamer presentation for paper X." · "Read `./papers/foo.pdf` and generate a detailed slide deck." · "Make a presentation-ready paper reading deck for this paper." → a detailed XeLaTeX Beamer deck under `slides/<paper-short-name>/`.

**Multi-paper examples:** "Analyze these papers and find research gaps." · "Compare paper A, B, and C." · "Help me understand what has been done and what remains open." · "Find possible research directions from these papers." → a research-gap memo under `analysis/<topic-short-name>/` **plus** a synthesis presentation deck under `slides/<topic-short-name>-gap-analysis/` (skip the deck only if the user asks for memo-only).

If the user only asks for slides for a single paper, do not run a gap analysis.
A gap analysis produces one **synthesis** deck for the whole set (not one deck per paper); to get individual per-paper decks the user must ask for single-paper slide mode.
When several single-paper decks are requested in slide mode, work **pilot-first**: produce one complete, inspected deck, report, then continue only if asked.

## Starting a new workspace

The skills operate inside a paper-reading **workspace** (with `papers/`, `slides/`, `analysis/`, and the bundled Simple Beamer theme).
To start one:

```bash
cp -r workspace-template ../my-topic-workspace
```

Then drop PDFs in `papers/` and invoke a skill.
See [`workspace-template/AGENTS.md`](workspace-template/AGENTS.md) for how the skills are used from inside a copied workspace.

## Global principles (apply to both skills)

- **Understand before writing.** For each paper, identify title, authors, venue/year, abstract, problem, motivation, contribution, method, implementation, evaluation setup, key results, limitations, related work, implications.
  If a paper can't be fully read, record the limitation — do not pretend, and do not invent details.
- **Claim–evidence discipline.** Ground every nontrivial claim in the paper (section / figure / table / page).
  Use careful language for inferences; mark uncertain claims as uncertain; do not overclaim that a research gap is novel without evidence.
- **Critical reading.** Distinguish what a paper *claims* from what it *demonstrates*, the assumptions its evaluation depends on, what is unproven, and what may not generalize.
  Explain both the contribution and its boundary.
  Be respectful but critical.
- **Terminology consistency.** Define terms once and use them consistently; when papers use different names for similar concepts, normalize them explicitly.
- **Outline first.** Produce a structured outline before writing the final deck or memo.

## What not to do

Do not: delete or modify original papers; move or modify the Simple theme (`beamerthemeSimple.sty`); place generated slides outside `slides/` or gap analyses outside `analysis/`; silently skip papers; generate shallow summaries; copy large chunks of paper text; create text-only decks; use unexplained screenshots; hide compilation failures; claim a deck is presentation-ready if it does not compile or if the PDF has not been visually inspected; invent paper details or citations; or overclaim novelty without evidence.

## Final quality bar

The result should feel like a careful graduate student read the paper(s), understood them, prepared teaching materials or a research memo, and thought critically about what the work means.
For decks: good enough to support a real graduate-level oral presentation.
For gap analysis: enough to help a researcher understand the landscape, avoid weak directions, and identify concrete next research questions.

---
> Source: [Haouo/paper2beamer-skills](https://github.com/Haouo/paper2beamer-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
