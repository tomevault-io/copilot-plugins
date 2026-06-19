## interview-prep-template

> A personal interview prep wiki on a three-layer system (Karpathy's pattern):

# AGENTS.md — Interview Prep Wiki

## Purpose

A personal interview prep wiki on a three-layer system (Karpathy's pattern):

- **Layer 1 — `sources/`**: raw material you drop in (mock-interview recordings & transcripts, real-interview & career-chat notes, reading, project descriptions). Content is immutable — you own it, I never edit it. An unprocessed file lives directly in its source folder (e.g. `sources/projects/`); once I've mined it, I move it into that folder's `processed/` subfolder.
- **Layer 2 — `wiki/`**: LLM-maintained markdown synthesizing the sources into reusable answers, frameworks, and concepts. I (the LLM) own this entirely.
- **Layer 3 — this file**: how I operate the wiki.

The loop: you drop materials into `sources/`; I lift the durable content into `wiki/` and move the source into `processed/`; you query me and I answer from the wiki. On `lint` I flag gaps and stale claims. The wiki is the persistent, compounding artifact.

Targets: interviews for **[replace with your target functions — e.g. product management, GTM strategy & operations, engineering]** across **[N]** companies.

## Directory layout

```
sources/                      ← Layer 1 (raw material — content is immutable)
  recordings/                   audio/video of mock interviews       (drop unprocessed files here)
    processed/                    move here once mined
  transcripts/                  verbatim text from mock recordings   (drop unprocessed files here)
    processed/                    move here once mined
  interview-notes/              notes/recall of an interview or chat (drop unprocessed files here)
    processed/                    move here once mined
  reading/                      articles, book chapters, AI news     (drop unprocessed files here)
    processed/                    move here once mined
  projects/                     user's detailed project descriptions (drop unprocessed files here)
    processed/                    move here once mined

wiki/                         ← Layer 2 (you own)
  companies/<company>/
    overview.md                 strategy, products, recent moves
    <role>.md                   role-specific JD, role evaluation framework
  behavioral/<question>.md      one file per potential behavioral question
  frameworks/<name>.md          reusable analytical templates (GTM, AI transformation, prioritization, etc.)
  ai-knowledge/<name>.md        AI domain knowledge
  interview-debriefs/<role>/
  <round>-<date>-<role>.md      per-interview debriefs

AGENTS.md                     ← this file
index.md                      ← catalog of every wiki page
log.md                        ← chronological record of ingests, lints, structural changes
```

## Conventions

### Naming
- All file and folder names: lowercase, kebab-case (`tell-me-about-conflict.md`, not `Tell Me About Conflict.md`).
- Interview debriefs: `<round>-<YYYY-MM-DD>-<role>.md`, inside a per-role folder `wiki/interview-debriefs/<role>/`.
- Behavioral question slugs capture the **theme**, not the exact wording. Common phrasings live inside the file as a list. Example: `handling-conflict-with-peer.md` covers "tell me about a time you disagreed with a coworker," "describe a conflict you resolved at work," etc. When a new interviewer phrases the same theme differently, add their phrasing to the file's list — do not create a duplicate file.

### Cross-references
- Use `[[file-slug]]` wikilinks in markdown bodies. Link liberally.
- A `[[slug]]` that doesn't match an existing file is a TODO, not an error — it marks something worth writing later.
- Obsidian renders these natively.

### Frontmatter

Every wiki page starts with YAML frontmatter:

```yaml
---
name: file-slug
description: One-line summary
type: behavioral | framework | ai-knowledge | company-overview | company-role | interview-debrief
sources: [project-slug, transcript-slug]    # source files this draws from
related: [other-page-slug]                   # related wiki pages
updated: YYYY-MM-DD
---
```

Source files in `sources/` get lighter frontmatter:

```yaml
---
title: Source title
date-of-input: YYYY-MM-DD
type: recording | transcript | interview-notes | reading | project
---
```

### Behavioral Pages: 

 Focused on a behavioral concept that will be tested in interviews (e.g., handling tough stakeholders). It holds one example initially and accumulates more examples as projects are processed — it does not fork into near-duplicate pages.

**Single example** (the initial state of a freshly-created page):

```
## STAR — concise (~90s spoken)
## Expansion points
## How to improve this example further?
```

**Multiple examples** (after subsequent ingests):

```
## Choosing which example to use
[brief decision rule — e.g., which interviewer profile, which competency dimension each example illustrates best, which is the safer default]

## Primary example: [[project-slug]] — [one-line angle]
### STAR — concise (~90s spoken)
### Expansion points
### How to improve this example further?

## Secondary example: [[other-project-slug]] — [different angle]
### STAR — concise (~90s spoken)
### Expansion points
### How to improve this example further?

and so on

```

### Framework Pages

Focused on How to think about solving GTM or Product problems in a structured manner? For example: Framework for 0 to 1 GTM or Framework for developing pricing models. 

**Single example** (the initial state of a freshly-created page):

```
## Framework - the industry standard for approaching this problem
## STAR - how I demonstrated that in the project (concise)
## Expansion points
## How to improve this example further?
```

**Multiple examples** (after subsequent ingests):

```
## Framework - the industry standard for approaching this problem

## Choosing which example to use
[brief decision rule — e.g., which interviewer profile, which competency dimension each example illustrates best, which is the safer default]

## Primary example: [[project-slug]] — [one-line angle]
### STAR - how I demonstrated that in the project (concise)
### Expansion points
### How to improve this example further?

## Secondary example: [[other-project-slug]] — [different angle]
### STAR - how I demonstrated that in the project (concise)
### Expansion points
### How to improve this example further?

and so on
```

### Interview Debrief Pages

Focused on capturing a comprehensive debrief from the interview for a specific role. Identify the question asked, followed by answer to that question, followed by a score of how well I answered that question, followed by how can I improve my answer further

```
## Question asked
## My answer
## Identified gaps
## How to improve my answer for next time?
## Score my answer against relevant evaluation framework for this role

## Question asked
## My answer
## Identified gaps
## How to improve my answer for next time?
## Score my answer against relevant evaluation framework for this role

## Question asked
## My answer
## Identified gaps
## How to improve my answer for next time?
## Score my answer against relevant evaluation framework for this role

and so on
```

## Operations

### Core principle: update before create

When ingesting any source, first find concepts that already exist and default to **updating existing pages before creating new concept pages** — the wiki compounds. A behavioral page holds multiple project examples; a framework page holds multiple concrete examples and grows new dimensions as projects reveal them. If a new source answers an existing theme — even differently — add it as another example (see the Behavioral and Framework page templates above) and keep the original; different projects illuminate different facets. Only create a new page if the theme has no existing home. This applies to every `ingest-*` operation.

**Coherency pass before any create.** Verify (a) no existing page already covers the theme, even under a different slug — search by *meaning*, not slug — and (b) the new page doesn't duplicate or contradict existing claims.

When an update makes one page diverge from a claim elsewhere:
- **Minor reconciliation — fix silently in-pass.** Wording alignment, more precise restatement, smoothing terminology.
- **Real contradiction — surface, don't auto-fix.** When a claim actively conflicts with another, or new evidence invalidates a prior conclusion, present it to the user in the response: what each side claims and which pages hold them; *why* they conflict; and 2–3 reconciliation options (narrow one to a context, deprecate one, merge into a synthesis, or keep both as alternatives to pick between per interview). Wait for the user's call. Mere divergence in *emphasis* is not a contradiction — handle it in the multi-example structure.

### `ingest-project`
Trigger: a markdown file appears directly in `sources/projects/` (not its `processed/` subfolder).

1. Read the project file end-to-end.
2. Identify STAR elements (situation, task, action, result) and the competency dimensions it exhibits, **using only what actually happened**. If the file contains an idealized/aspirational section (framed as "what I'd do differently," "the better version of this story," or similar), **exclude it from STAR answers** — interview answers must be factually true to what you did. Idealized content may live in a separate "How to improve this example further" section (useful for explicit follow-ups), but never silently merge into the primary answer.
3. For each behavioral question the project could answer, apply update-before-create + the coherency pass (see Operations preamble): extend the matching existing page with this project as another example, or create a new page only if no theme home exists. Add this project to the page's `sources:` list.
4. If it illustrates a framework, add it as a concrete example in the relevant `wiki/frameworks/<name>.md` (extend the framework if the project reveals a new dimension; only create a new framework page if the pattern has no home).
5. Update `index.md`.
6. Append a `log.md` entry.
7. Move the file: `sources/projects/` → `sources/projects/processed/`.

### `ingest-reading`
Trigger: an article, book chapter, or news clip appears directly in `sources/reading/` (not its `processed/` subfolder).

1. Read the source.
2. Extract concepts, frameworks, and trend signals.
3. Update or create relevant `wiki/ai-knowledge/` and `wiki/frameworks/` pages.
4. Update `index.md`. Append log entry.
5. Move file: `sources/reading/` → `sources/reading/processed/`.

### `ingest-recording`
Trigger: an audio/video file appears directly in `sources/recordings/` (not its `processed/` subfolder) — a **mock** interview. Recording is the *only* path that produces a transcript; real interviews and recruiter/advisor chats are never recorded — they're captured as notes.

1. **Transcribe** using local Whisper (see Tooling). Do not silently fail — if Whisper or ffmpeg is missing, surface that to the user.
2. Convert Whisper's `.txt` output into the durable transcript: write `sources/transcripts/<same-stem>.md` with frontmatter (`title`, `date-of-input`, `type: transcript`) followed by the verbatim transcript body, then delete the intermediate `.txt`. The transcript artifact is always markdown, never `.txt`.
3. Move the original audio: `sources/recordings/` → `sources/recordings/processed/`.
4. Run `ingest-transcript` on the new transcript.

### `ingest-transcript`
Trigger: a verbatim transcript of a **mock** interview exists directly in `sources/transcripts/` (not its `processed/` subfolder) — produced by `ingest-recording`, or pasted by you. (For a **real** interview you couldn't record, use `ingest-interview-notes` instead — those are notes/recall, not a verbatim transcript.)

1. Read the transcript end-to-end.
2. **Identify every question the interviewer asked.** For each, classify and route (applying update-before-create + the coherency pass throughout):
   - **Behavioral**: find the matching `wiki/behavioral/` page by meaning. If it exists, compare the spoken answer to the file's current best — fold in improvements if the spoken answer is stronger; if weaker, leave the best alone but record the gap in the debrief section for the role. If none exists, create one, drafting from the spoken answer (or, if it was weak, an improved version built on the projects in `sources/projects/processed/`).
   - **Case**: identify (or create) the applicable `wiki/frameworks/` page; update it if the transcript reveals a stronger pattern.
   - **Factual/ AI knowledge**: update or create the relevant `wiki/ai-knowledge/` page.
   - **Company-specific**: update the relevant `wiki/companies/<co>/<role>.md` file or `overview.md` file if new information was discovered about the company.
3. **In the debrief, always flag** which answers were preserved (yours was strong), improved (yours was weak), and which behavioral questions were new additions.
4. Write a debrief `wiki/interview-debriefs/<role>/<round>-<YYYY-MM-DD>-<role>.md`: frontmatter (date, company, role, interviewer names if known, score); follow the structure laid out in the Interview Debrief Pages section above (per question: question asked, my answer, identified gaps with specific improvement actions, and **a score out of 5** against the role's evaluation framework — rubric TBD, define collaboratively with the user the first time this runs; see Open items).
5. Update `index.md`. Append log entry. Move the transcript: `sources/transcripts/` → `sources/transcripts/processed/`.

### `ingest-interview-notes`
Trigger: a file exists directly in `sources/interview-notes/` (not its `processed/` subfolder) — your notes on, or memory of, a **real** interview you couldn't record. (If it wasn't an evaluative interview — e.g. a recruiter or advisor chat — use `ingest-non-interview-conversation` instead.)

Process it **exactly like `ingest-transcript`** — identify every question, route each (behavioral / case / factual / company-specific) by meaning with update-before-create + the coherency pass, then write a scored debrief — with three differences:
1. The content is your **recollection**, not a verbatim transcript: treat any quotes as approximate, and where memory is fuzzy, note the uncertainty in the debrief rather than inventing detail.
2. The debrief reflects a **real** interview's performance — often the highest-signal kind to learn from.
3. Move the file: `sources/interview-notes/` → `sources/interview-notes/processed/`.

### `ingest-non-interview-conversation`
Trigger: your **notes on** a **recruiter chat, advisor conversation, or career discussion** — NOT a formal interview. Like real interviews, these aren't recorded — you capture them from notes or memory, so they live in `sources/interview-notes/` alongside real-interview notes.

**How to distinguish from `ingest-interview-notes`**: in an interview, the interviewer asks questions and evaluates your answers. In a non-interview conversation, you're mostly *receiving* information (about a role, hiring process, market, team) or *getting advice* — there's no answer-scoring relationship. When ambiguous, ask the user before processing.

1. Read the notes end-to-end.
2. Mine for durable content. There are no answers to score and no behavioral questions to file. Instead:
   - **Company / role information** (responsibilities, hiring bar, interview process, panel composition, comp signals, team culture, recent moves the counterparty shared): update or create `wiki/companies/<co>/overview.md` or `<role>.md`.
   - **POVs and signals from the counterparty** (what they emphasized, deflected, seemed to care about): file inside the relevant company/role page, or `wiki/ai-knowledge/` if broader.
   - **Action items for the user** (things they were advised to prepare, study, send, or follow up on): surface in the response — do not silently file these.
3. Do NOT produce a scored debrief. The 5-point framework is for interview performance, not conversations.
4. Update `index.md` if new pages were created. Append log entry.
5. Move the notes file: `sources/interview-notes/` → `sources/interview-notes/processed/`.

### `query`
Trigger: the user asks a question.

1. Read `index.md` first to locate relevant pages.
2. Read the targeted wiki pages.
3. Answer using their content; cite the pages by `[[slug]]`.
4. If the answer synthesizes something new worth keeping, offer to file it as a new wiki page (usually in `frameworks/` or `ai-knowledge/`). Don't create silently — propose first.

### `lint`
Trigger: the user explicitly asks for a lint pass.

1. Walk `wiki/` and flag:
   - Pages with `updated` older than 7 days for `companies/` (targets move fast), 30 days for `ai-knowledge/` (knowledge drift).
   - `[[wikilinks]]` pointing to non-existent files.
   - `[[wikilinks]]` placeholders that should now become real pages.
   - Behavioral questions without a strong answer, or without any `sources:` link.
   - Companies without `overview.md`.
   - Interview debriefs without a score.
2. Walk `sources/` and flag unprocessed files (sitting directly in a source folder, not its `processed/` subfolder) older than 7 days = dropped balls.
3. Produce a punch list. **Do not auto-fix.** The user decides which items to tackle.

## Tooling

### Transcription (local, private, free)
- **Tool**: `openai-whisper`, callable as `whisper`. Runs entirely locally — no audio leaves the machine. Requires `ffmpeg` on PATH. (Optional — only needed for `ingest-recording`. See the README for install pointers.)
- **Default model**: `medium.en` — interview-grade accuracy, near-real-time on CPU.
- **Command pattern** (the UTF-8 env var avoids a Windows cp1252 crash on non-ASCII output):
  ```
  PYTHONIOENCODING=utf-8 whisper "<audio-file>" --model medium.en --output_format txt --output_dir sources/transcripts/
  ```
  Whisper has no native markdown output, so it writes plain `.txt`. That `.txt` is an **intermediate only** — `ingest-recording` step 2 wraps it into `<stem>.md` with frontmatter and deletes the `.txt`. The durable transcript artifact is always the `.md`.
- If `whisper` or `ffmpeg` is missing, surface the missing dependency. Do not silently fail.

## Index and log

- **`index.md`** — catalog of every wiki page with a one-line description, grouped by folder. Read this FIRST when answering a query so you don't miss relevant pages.
- **`log.md`** — append-only chronological record. Newest at the bottom. Format:
  ```
  ## [YYYY-MM-DD] <op-type> | <subject>
  One-paragraph note.
  ```
  Line-prefix parseable by standard tools.

## Ownership boundaries

You (the LLM) own:
- All of `wiki/`
- `index.md` and `log.md`
- File *movement* from a source folder into its `processed/` subfolder in `sources/`

You don't own:
- The *content* of any file in `sources/` (the user's words — immutable)
- This operating manual (`CLAUDE.md` / `AGENTS.md`) schema — propose changes, don't silently rewrite

## Open items

- **Scoring rubric** — define a 5-point interview-debrief rubric collaboratively with the user before the first real debrief. A practical starting point: derive the competency dimensions from one of your own processed project files (e.g. a competency self-assessment), then reuse that rubric across every debrief so scores are comparable.
- **Privacy** — `sources/` is git-ignored by default (raw, private, large). Your `wiki/`, `log.md`, and `index.md` will accumulate real, sensitive content (company names, interviewer names, candid self-assessments). Keep this repo **private** if you commit them. See `.gitignore` and the README.

---
> Source: [AbhiK189/interview-prep-template](https://github.com/AbhiK189/interview-prep-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
