## acelerado-desafios

> This repo hosts monthly programming challenges.

# Writing problem statements

This repo hosts monthly programming challenges.
Each challenge lives in its own dated folder (`YYYY-MM-slug/README.md`) and the `README.md` *is* the problem statement - it's what a contestant reads to decide whether to attempt the challenge and how to attempt it.

This file documents the writing style for those problem statements.
It does not cover code style or repo tooling.

## Tone

Write like a knowledgeable friend explaining the problem, not like a textbook or a spec.
Conversational, opinionated, direct.
A reader should finish the framing and feel "ok, I get why this is a real thing," not "ok, here are the requirements."

Do not use slang and do not try to be funny.
This applies especially when writing in Portuguese (or any non-English language): Claude tends to land slang and humor poorly outside English, producing text that reads as cringe or off-key to native speakers.
Plain, clear, slightly opinionated prose travels much better.
"The cheap autofocus misses focus on most frames" is fine; reaching for slangy phrasings like "o autofoco do bagulho" is not.

Be concrete.
Name specific hardware, specific numbers, specific failure modes.
"A cheap CMOS sensor with bad continuous autofocus" beats "a low-quality camera."
But concrete details serve the framing - they are not the framing.

Avoid the "you bought X" / "imagine you are Y" personal-anecdote opener.
It collapses the problem to one person's story and makes the challenge feel like a toy exercise.
Land the reader in a *family of real problems* instead - see the opener structure below.

## Opener structure

The first section (typically `## O cenário` or `## The scenario`) should zoom in across three paragraphs before any spec or task statement appears:

1. **Wide-angle hook.** Why does this *kind* of problem exist in the world? Historical or cultural framing the reader recognizes immediately. (E.g. "photography has always driven software and algorithm development.")
2. **The gap.** What's the constraint that creates the problem? Where does the idealized solution break down, and for whom? (E.g. "smartphones spoiled us, but cheap drones can't carry that hardware.")
3. **The concrete failure mode.** Land on the specific symptom this challenge attacks, with the physical or algorithmic mechanism behind it. This is where the technical hooks the reader will need (the noise model, the failure distribution, the corruption process) get named.

Immediately after the three paragraphs, show a **visual comparison** - input vs. expected, before vs. after, broken vs. working - so the reader's eye sees the problem before reading the spec.

## Section order after the framing

1. `## A tarefa` / `## The task` - one-sentence statement of what the contestant must produce, plus the ranking metric in plain language.
2. `## Spec` - I/O format, harness invocation, validation caps, ranking and tiebreakers. This is the contract; be precise.
3. `## Como o <thing> foi gerado` / `## Forward model` - when the test data is synthetic, document the generation process exactly. Contestants should be able to (and are expected to) exploit this.
4. `## A referência` / `## The baseline` - the dumb reference solver, what it does, and *why it is intentionally dumb* so contestants understand what "doing better" looks like. List a few directions to attack it.
5. `## Dataset público` and `## Dataset oculto` - what's in each, and an honest statement of whether the hidden set is "more of the same" or has gotchas.
6. `## Como testar localmente` - copy-pasteable commands.
7. `## Como submeter` - point to the generic `SUBMISSION.md` for the boilerplate, and call out anything challenge-specific.
8. `## Atribuição` - credits for datasets, libraries, papers.

Sections after the framing can be reordered or merged when it makes sense, but the framing -> task -> spec -> forward model -> baseline progression should hold: each section answers a question the previous one raises.

## Formatting

Markdown prose: break at sentence boundaries, one sentence per line, paragraphs separated by blank lines.
Do not hard-wrap at 80/90 columns.
Code blocks, tables, and fenced content are exempt.

Use **bold** for the technical hooks the reader should walk away knowing - the names of the failure mode, the metric, the cap.
Don't bold for general emphasis; reserve it.

Use tables for input/output comparisons and dataset listings.
Use fenced code blocks (with language tag) for harness invocations and reference snippets.

When citing files in this repo, use relative links from the challenge folder (`[reference/wiener.py](reference/wiener.py)`), not absolute paths.

### ASCII-only punctuation in prose

Use plain ASCII for punctuation that has a non-ASCII "smart" or typographic variant.
Some editors, terminals, and downstream tooling render non-ASCII punctuation poorly, and it tends to leak in unintentionally from autocorrect or LLM output.

Concrete substitutions:

- em-dash and en-dash (`—`, `–`) -> hyphen `-`
- ellipsis (`…`) -> three dots `...`
- arrow (`→`) -> ASCII arrow `->`
- curly quotes (`“ ” ‘ ’`) -> straight quotes `" '`
- multiplication sign (`×`) -> letter `x`

Code blocks, file paths, and quoted external text are exempt - keep those verbatim.

## What to leave out

No marketing language ("blazingly fast," "state-of-the-art," "cutting-edge").
No motivational filler ("good luck and have fun!").
No "in this challenge, you will learn..." - contestants are not students, and the framing already communicates the territory.

Don't restate the spec inside the framing or vice versa.
The framing tells the reader *why*; the spec tells them *what exactly*.
Mixing them dilutes both.

---
> Source: [wainejr/acelerado-desafios](https://github.com/wainejr/acelerado-desafios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
