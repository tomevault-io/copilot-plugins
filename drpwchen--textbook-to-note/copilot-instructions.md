## textbook-to-note

> You are helping your user set up `textbook-to-note`: a local-first pipeline

# AGENTS.md — Instructions for your AI coding agent

You are helping your user set up `textbook-to-note`: a local-first pipeline
that converts their own PDF/EPUB textbooks into searchable markdown, then
into structured, fully-cited notes in their personal knowledge vault
(Obsidian, Logseq, or a plain markdown folder). This repository is designed
to be deployed **by you**, an AI coding agent, working directly with the
user rather than requiring them to hand-run every script themselves.

Read this file fully before doing anything. Then work through the steps
below in order, checking in with the user at the marked decision points.

## What you're setting up

```
converter/    — PDF/EPUB → markdown conversion (0 LLM tokens)
figures/      — on-demand figure extraction with QC gating
skills/       — two Claude Code skill definitions (drop-in to ~/.claude/skills/)
workflows/    — the note-writing workflow specification
templates/    — real production note templates (zh-TW + English) for Step 1.1's topic-type table
docs/         — architecture + OCR-ladder reference docs
examples/     — one example output note showing the target format
shared/       — shared config (paths, env var names)
requirements.txt
```

Read `docs/architecture.md` first for the full picture, and
`docs/ocr-ladder.md` if you'll be doing any OCR-heavy conversions.

## Step 1: Understand the user's situation

Ask, or infer from context:
- Where do their textbook PDFs/EPUBs already live? (a folder, a cloud-synced
  drive, etc.)
- What notes tool do they use, and where is their vault/notes folder?
- Do they want the optional semantic search index (LanceDB + a local
  embedding model via ollama), or is grep-only fine for their corpus size?
  Semantic search pays off once there are more than a handful of books;
  for a small personal library, skip it initially and add later.
- Do they have a GPU available locally? This affects whether the OCR ladder
  (`docs/ocr-ladder.md`) can use a GPU-accelerated engine or should fall
  back to CPU-only options. See "Choosing your hardware tier" in
  `docs/ocr-ladder.md` for the tier table (CPU-only / Apple Silicon 8GB /
  Apple Silicon 16GB+ / NVIDIA 8GB / NVIDIA 16GB+) before recommending any
  OCR/vision-QC/embedding stack.

## Step 2: Install dependencies

```bash
pip install -r requirements.txt
```

`requirements.txt` covers PDF parsing, EPUB conversion (needs `pandoc` on
`PATH` separately — check with `pandoc --version` and prompt the user to
install it if missing), and the optional semantic-search stack. If the user
declined semantic search in Step 1, you can skip installing that subset.

## Step 3: Configure paths

Configuration is environment-variable driven with repo-relative defaults —
`shared/config.py` documents every variable and works with zero setup:
drop books in `./books`, get markdown in `./output`.

Set env vars (in the shell, or a `.env` you source) only where the defaults
don't fit:
- `BOOKS_DIR` — where the user's PDFs/EPUBs live (default `./books`)
- `OUTPUT_DIR` — where converted markdown goes (default `./output`; keep it
  outside the notes vault — this corpus is for your reference, not for the
  user to read directly)
- Optional OCR fallback: `SURYA_VENV_PY` + `SURYA_ADAPTER` (see
  `docs/ocr-ladder.md`)
- Optional semantic search: `INDEXER_SCRIPT` + `VAULT_SEARCH_DIR`
- Figure output/cache locations: env-driven constants documented at the top
  of `figures/figure_qc_gate.py` (default `./output/figures`, inside the
  user's vault attachments folder if they want embeds to resolve)

Run `python shared/config.py` to print the resolved configuration and
confirm it before converting anything. Never hardcode paths into scripts or
skill files — always go through `shared/config.py` or env vars, so the repo
stays portable across machines and users.

## Step 4: Convert a first book (smoke test)

Pick one book the user cares about and convert it end to end, narrating what
you're doing:

```bash
python converter/convert.py "path/to/one.pdf" --book-label "Author Title — Ch.1"
```

Check the output markdown for:
- Readable, non-garbled text
- `<!-- page N -->` markers present
- Any `<!-- REF: Fig. X.Y → ... -->` markers where the source mentions
  figures

If the text looks garbled or mostly empty, read `docs/ocr-ladder.md` and
re-run with the OCR-forcing flag (`--force-surya`, requires the optional
OCR engine from `docs/ocr-ladder.md`) rather than assuming the conversion is
broken — most garbled output on a first try is a silent fitz failure that
the OCR ladder is built to catch.

Once the smoke test looks right, convert the rest of the user's priority
books:

```bash
python converter/convert.py --batch-dir "path/to/their/textbook/folder"
```

This can take a while for a large library — run it as a background job and
report progress, don't block the conversation on it. Don't run two batch
conversions against the same `OUTPUT_DIR` concurrently — progress tracking
is last-writer-wins, so overlapping runs will corrupt each other's progress
state.

## Step 5: (Optional) Build the semantic index

Only if the user opted in during Step 1:

```bash
python converter/post_convert.py --index
```

Requires the local embedding model to be running (e.g. `ollama pull
bge-m3` then `ollama serve`, or however their local inference server is
set up). Verify the index actually returns results for a test query before
telling the user it's ready.

## Step 6: Install the two Claude Code skills

```bash
mkdir -p ~/.claude/skills
cp -r skills/textbook-to-md ~/.claude/skills/
cp -r skills/figure-remap ~/.claude/skills/
```

Each copied `SKILL.md` references its scripts as `{REPO}/figures/...` or
`{REPO}/converter/...` — the scripts themselves stay in this cloned repo,
only the `SKILL.md` files move. Open each copied `SKILL.md` and replace every
`{REPO}` placeholder with the absolute path of this clone (e.g.
`C:\Users\you\textbook-to-note` or `/home/you/textbook-to-note`), so the
example commands resolve to the actual `converter/` and `figures/` code.

## Step 7: Run the note-writing workflow

`workflows/note-writing.md` is the full specification for turning a
converted textbook chapter into a structured note. It is written generically
so you should adapt phases 2/3 (the optional pluggable enrichment stages) to
whatever domain-specific tools the user actually has — a clinical evidence
API, a regulations database, a literature search tool, or nothing at all if
their domain doesn't need it.

Before writing the user's first real note, walk through
`examples/example-note.md` with them so they can confirm the target format
(nested bullets, per-claim citations, figure embed style) matches what they
want before you commit to writing dozens of notes in that shape.

Then, for a real topic:
1. Follow Phase 1 → 1.5 as written — draft blind from the textbook corpus,
   skeleton before prose
2. Skip phases 2/3 unless the user has a relevant enrichment source
   configured
3. Run Phase 3.5 figure harvest through the `figure-remap` skill — never
   hand-crop images yourself, always go through the QC-gated entrypoint
4. Run Phase 4 to merge with any existing note on the topic, following the
   deconstruct-and-reslot approach — do not just append new content to old
5. Optionally run Phase 5 link suggestion

## Step 8: Verify the note-format hook (optional but recommended)

If you (the agent) or the user want a mechanical pre-flight check before
every note write — catching missing citations, unescaped table syntax,
widthless figure embeds, and similar structural issues — build a small
format-lint script and wire it in as a pre-write hook in your agent
framework. This repo doesn't ship one by default since notes-tool
conventions vary too much across users, but `workflows/note-writing.md`'s
self-check section lists exactly what such a hook should catch.

## Token guardrails

These apply any time you're working with the converted corpus, not just
during initial setup:

- When consulting the converted corpus, **always grep or semantic-search
  first**, then `Read` only a bounded window (roughly ≤150 lines) around the
  hit. Never `Read` an entire `full_text.md` or a whole chapter file into
  context — a single book can be well over 1M tokens, and reading it whole
  defeats the entire point of converting to a searchable corpus.
- Frontier-vision figure escalation (ladder step 5 in `docs/ocr-ladder.md`)
  is per-figure opt-in and capped — at most a small fixed number of
  escalations per note. Prefer leaving a `<!-- TODO: figure -->` placeholder
  over a third escalation on the same note, and never batch-escalate a set
  of figures to frontier vision at once.

## Ongoing use

Once set up, the day-to-day loop is:
1. New textbook arrives → `converter/convert.py` (or batch-dir for several
   at once)
2. User wants a note on a topic → run the note-writing workflow
3. Note needs a figure mid-conversation, outside the full workflow → call
   `figure_remap.py extract` directly per `skills/figure-remap/SKILL.md`

## Guardrails — read before touching the filesystem

- Never delete or move files outside this repo and the user's configured
  vault/attachments directory without explicit confirmation.
- Never use raw destructive file operations (recursive delete, force move)
  on the user's attachments folder from within an automated sub-step —
  destructive operations belong to the orchestrating agent only, with the
  user able to see and approve what's being removed.
- Treat the user's source PDFs as read-only source-of-truth; all pipeline
  output should be a fresh copy, never an in-place edit of the original.
- If a step in this file references a script or path that doesn't exist yet
  in this repo, say so explicitly rather than guessing at its interface.

---
> Source: [drpwchen/textbook-to-note](https://github.com/drpwchen/textbook-to-note) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
