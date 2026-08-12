## ankihelper

> Turns each week's Monash Master of AI content into a high-quality, ready-to-import Anki deck, doing as much as possible without your input. Only genuine tradeoffs come back to you.

# Anki Card Pipeline

Turns each week's Monash Master of AI content into a high-quality, ready-to-import Anki deck, doing as much as possible without your input. Only genuine tradeoffs come back to you.

## Default: flag it, don't drop it

A card you don't want costs a few seconds to delete in review. A concept that never got flagged costs a gap you won't discover until it's tested — you can't reject a card you never saw. So: silent exclusion needs high confidence something's genuinely not content (agenda slides, "any questions?", housekeeping). Anything else with real substance and any real doubt — unclear exam-relevance, a possible duplicate, procedural content that might carry a concept — gets fully drafted and flagged, never just described. A long flagged list some week means the material was ambiguous, not that something's wrong; don't quietly raise the bar to shrink it. An essay-framed topic still yields cards for any named terms or definitions inside it.

## Folder structure

```
/units/<UNIT_CODE>/
  textbooks/
    <book>.pdf             — added once, used across all weeks for this unit
    <book>-toc.md          — chapter -> page-range index (built once per book, see below)
    page-cache/<book>/     — cached rasterized page ranges
  week-00/
    instructions.txt       — per-week instructions: what each file is, page ranges, textbook refs
    objectives.md          — this week's learning outcomes (required)
    <anything>.pdf         — lecture notes, lab sheets, or any other materials, named however
    page-images/<stem>/    — cached page rasterizations, created as needed
  week-01/
    ...
/output/
  <UNIT_CODE>-week<NN>.txt         — the deck (format in docs/anki-format.md)
  <UNIT_CODE>-week<NN>-flagged.md  — fully-drafted cards held for review, one reason each
/concepts-seen.md   — one line per concept already carded, shared across ALL units
docs/anki-format.md — exact output syntax, tags, worked examples — open at step 5, not before
scripts/            — helper scripts
```

## instructions.txt

Read this before any other file in the week folder. It tells you what each file is and how to treat different parts of it. Written by the human in plain language — interpret it naturally. Examples of what it might contain:

```
week3-slides.pdf pp 1-18 are pre-reading, pp 19-40 are lecture, pp 41-45 are homework
lab.pdf is the lab sheet
textbooks/bishop-prml.pdf Chapter 4
ignore the last 3 slides
```

If instructions.txt is empty or absent, infer content type from filenames and content as best you can.

## Workflow

1. Read `instructions.txt` — use it to interpret everything else in this folder.
2. Read `objectives.md` — treat it as the spec.
3. Read all other files in the week folder. For every PDF, rasterize and view pages visually (see Reading PDFs) — never rely on text extraction alone.
4. For any textbook reference in instructions.txt, resolve and extract the relevant pages (see Textbook chapters).
5. Read `/concepts-seen.md` for what's already covered.
6. Draft cards, sorting by the default above.
7. Open `docs/anki-format.md` and write the deck file(s) and `flagged.md` in that format.
8. Append newly-carded concepts to `/concepts-seen.md`, including anything approved from a previous flagged list.
9. Report card count, objective coverage (flag any objective whose only coverage is a still-pending flagged item), and the flagged list. Any reply naming or numbering something — from the list or anywhere else in the material — gets carded immediately. These rules are defaults for your own judgment, not a veto on an explicit instruction.

## Python environment

A virtual environment lives at `.venv/` in the project root. Use `.venv\Scripts\python` when running any script — don't rely on the system Python:

```powershell
.venv\Scripts\python scripts\rasterize_pdf.py <pdf> page-images
.venv\Scripts\python scripts\textbook_chapter.py list-toc <book.pdf>
```

## Reading PDFs

Text extraction misses anything that's an image or vector drawing — formulas, architecture diagrams, plots, screenshots. A slide that's mostly a diagram can return two lines of text with no error and no sign anything was skipped. Don't treat a slide as content-free just because its extracted text is thin.

Before reading any PDF, rasterize every page to a PNG (`.venv\Scripts\python scripts\rasterize_pdf.py <pdf> page-images` — one page per image, ~200 DPI) and view each page image directly alongside whatever text extracts cleanly. Images are cached so a re-run doesn't re-convert.

## Textbook chapters

Books live in `units/<UNIT_CODE>/textbooks/`, never copied into a week folder — instructions.txt is the per-week pointer. Never rasterize or read a whole textbook for one chapter.

To resolve a reference: check `textbooks/<book>-toc.md` for the chapter's page range. If that file doesn't exist yet, run `.venv\Scripts\python scripts/textbook_chapter.py list-toc textbooks/<book>.pdf` — it prints the embedded outline if the PDF has one. If it doesn't, it rasterizes the first few pages so you can read the printed contents page and write `<book>-toc.md` yourself — one-time per book.

Once you have the page range, run `.venv\Scripts\python scripts/textbook_chapter.py extract textbooks/<book>.pdf <start> <end>` to rasterize just that range and view those pages the same way as slides.

## Card-writing rules

- One fact or idea per card. A list of five things is five cards, or one Cloze note with five deletions — never one card with a five-part expected answer.
- Exactly one unambiguous correct answer. Avoid "discuss" or "explain" prompts.
- Your own words on the card itself — don't mirror slide or textbook phrasing closely.
- Enough context to stand alone months later — name the algorithm/unit if the term is ambiguous elsewhere.
- Cloze for sequences/steps/multi-part lists. Basic for single facts, definitions, "X vs Y".
- For genuinely similar concepts (L1 vs L2, batch vs mini-batch vs stochastic), add a contrast card instead of two cards that blur together.
- Lab content: card the reasoning ("why normalise before PCA"), never the syntax.
- No raw newline or tab inside a field — use `<br>` for line breaks (matters most for multi-line Cloze).
- Tag every card `UnitCode::WeekNN::Topic`.
- 15–25 new cards/unit/week is a ceiling to justify exceeding, not a quota to fill — a light week producing 8 is fine. Doesn't limit what gets flagged.

## Learning your preferences over time

When a flagged-list reply reveals a pattern — you always want a certain category carded, or never want another — save that as a memory rather than re-flagging the same category next week. Calibration should tighten over the semester, not restart cold every session.

---
> Source: [LouisGenesi2/AnkiHelper](https://github.com/LouisGenesi2/AnkiHelper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
