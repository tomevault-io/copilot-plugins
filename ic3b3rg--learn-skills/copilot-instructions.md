## learn-skills

> A set of agent skills that package **evidence-backed pedagogical methodologies** (explain-and-check, quiz-me, connect-to-what-you-know, ask-me-questions, learn-by-doing, linked-notes, flashcards) as workflows applied to code. The motivation is anti-cognitive-surrender: closing the comprehension gap that opens between a human and a codebase when an LLM has done the work on the human's behalf.

# learn-skills

A set of agent skills that package **evidence-backed pedagogical methodologies** (explain-and-check, quiz-me, connect-to-what-you-know, ask-me-questions, learn-by-doing, linked-notes, flashcards) as workflows applied to code. The motivation is anti-cognitive-surrender: closing the comprehension gap that opens between a human and a codebase when an LLM has done the work on the human's behalf.

For the rationale, glossary, and decisions log, read [CONTEXT.md](CONTEXT.md). For the user-facing overview, read [README.md](README.md). For load-bearing decisions, read [docs/adr/](docs/adr/).

## Project structure

```
README.md            → User-facing overview
CONTEXT.md           → Design rationale, glossary, decisions log
CLAUDE.md            → This file: operating conventions for editing the project
docs/
  adr/               → Architectural decision records (numbered)
  sources.md         → Verification source policy (cited from every skill's principle 4)
skills/
  <skill-name>/
    SKILL.md         → Required, ≤ 100 lines
    <REFERENCE>.md   → Optional, format-spec / reference content, one level deep
```

## Skills (9 total, two families + two meta)

**Meta** — `start-learn`, `assess`
**Session methodologies (dialogic)** — `explain-and-check`, `quiz-me`, `connect-to-what-you-know`, `ask-me-questions`
**Generative methodologies (artefact-producing)** — `learn-by-doing`, `linked-notes`, `flashcards`

Bloom and SOLO are **editorial concepts**, not skills. They appear internally as a chooser and in `assess` as the output scale; they are not listed in the user-facing README.

## Conventions (non-negotiable)

- **SKILL.md ≤ 100 lines.** Hard cap. Claude previews via `head -100`; content past line 100 is invisible during discovery. Split overflow into a sibling reference file (`UPPER-KEBAB.md`).
- **YAML frontmatter**: `name` (kebab-case, ≤ 64 char) + `description` (≤ 1024 char, third person, what + when triggers).
- **Reference files one level deep only.** `SKILL.md` → `REFERENCE.md` is fine. `SKILL.md` → `REFERENCE.md` → `OTHER.md` is forbidden — Claude may not follow the second hop.
- **Five operating principles** apply to every skill (see [CONTEXT.md](CONTEXT.md)): (1) agent withholds; (2) student speaks first; (3) artefact is the judge; (4) source fidelity → [docs/sources.md](docs/sources.md); (5) exit is a transfer test. New skills must satisfy all five.
- **Anti-patterns use "because Y"** — `Don't X because Y tends to cause Z`. Bare prohibitions are weaker than explained ones.
- **English everywhere** in files. The agent translates user-facing dialogue at runtime — never write Italian (or any other language) in SKILL.md, REFERENCE.md, or docs.
- **Cite, never assert from memory.** Every claim about external API / library / framework behaviour requires a verifiable link to an authoritative source ([docs/sources.md](docs/sources.md)).

## Boundaries (what NOT to do)

- **Don't add scheduling methodologies** (spaced repetition, interleaving as skill/flag). Rejected in [ADR 0002](docs/adr/0002-no-scheduling-no-monolithic-scores.md) — temporal state management is a sub-project that would swallow this one.
- **Don't add monolithic numeric scores** to `assess`. Discrete Bloom/SOLO levels only. Same ADR.
- **Don't add an automatic router.** `start-learn` proposes; the user decides. Auto-routing violates principle 1.
- **Don't write SKILL.md in Italian** (or any non-English language). Bilingualism is a runtime behaviour, not a file format.
- **Don't promote Bloom or SOLO to invocable skills.** They are editorial concepts. See [ADR 0001](docs/adr/0001-evidence-based-core-and-editorial-concepts.md).
- **Don't add a new skill without evidence**. Methodologies in this repo are evidence-backed (explain-and-check, quiz-me, connect-to-what-you-know are the empirical core). Folk pedagogy doesn't enter.

## Authoring workflow

To create or modify a skill, use the `write-a-skill` skill (`/write-a-skill`). It enforces the SKILL.md template, line cap, and frontmatter conventions.

After any change to a SKILL.md, verify:

```bash
# Line cap (must be ≤ 100 for every SKILL.md)
for s in skills/*/SKILL.md; do wc -l "$s"; done

# Every skill links docs/sources.md from principle 4
grep -L "docs/sources.md" skills/*/SKILL.md   # must return nothing

# Frontmatter present
for s in skills/*/SKILL.md; do head -1 "$s"; done   # must all be "---"
```

---
> Source: [Ic3b3rg/learn-skills](https://github.com/Ic3b3rg/learn-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
