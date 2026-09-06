## glossary-term-template

> Glossary term pages — full article required for every term


# Glossary Term Template

Use when editing `data/glossary_terms.csv`, term generators, or `terms/` pages.

**Full source of truth:** `docs/glossary-term-template.md`  
**HTML diagrams (optional):** `docs/term-diagrams.md` — CSV `diagram_id` → `data/term_diagrams/{id}.json`  
**Editorial quality:** `docs/editorial-quality.md`  
**SEO shared rules:** `docs/seo-article-guidelines.md`

## Production standard

- **Every term** in `glossary_terms.csv` is a **full detail article** (not a one-line definition).
- Target volume: **300+ terms** (`validate_csv.py` warns below target).
- New row: `python3 tools/scaffold_glossary_term.py` — see `docs/glossary-term-template.md`.
- Bulk fill from practice DB: `tools/enrich_o4_glossary_details.py` → then manual review.
- Quality audit: `tools/audit_glossary_article_quality.py`.

## Required CSV columns (all terms)

Base: `term`, `category`, `tags`, `importance`, `short_def`, `definition`, `explanation`, `related_terms`, `legal_basis`.

Detail (mandatory — `tools/glossary_term_rules.py` / `validate_csv.py` **ERROR** if missing or too short):

| Column | Minimum |
|--------|---------|
| `article_title` | 10 chars |
| `article_lead` | 30 chars |
| `term_detail_body` | **180 chars**, **2+ paragraphs** |
| `exam_points` | **2+ items** (`;` separated, each 8+ chars) |
| `common_mistakes` | 20 chars |
| `memory_tip` | 15 chars |
| `example_question` / `example_answer` | 12 / 3 chars (○× or 5+ char answer) |
| `faq_1_*` 〜 `faq_4_*` | Q 6+ chars, A **100+ chars** (**4 required**) |
| `related_terms` | **2+ registered terms** (unregistered → **ERROR**, no link emitted) |

## Writing style (plain Japanese)

- Use **です・ます調** consistently; short sentences (40–60 chars).
- Explain jargon on first use; say **who / when / what** explicitly.
- Prefer everyday words over legalese (`及び`→`と`, avoid `当該`/`前述`/`において`).
- `validate_csv.py` **WARN** on readability anti-patterns in body/FAQ columns.

FAQ roles: (1) definition, (2) common mistake, (3) exam angle, (4) next study step.

`legal_basis`: optional; **WARN** if category is 法令・制度-like and `importance` is A/S but empty.

## Generated page structure

Same SEO skeleton as guide articles: lead → TOC → reliability → action box → numbered body (5–7 visible h2) → FAQ → basic info → official links → related terms → next pages.

Body sections from CSV: summary, exam points, definition (+ peer compare table from `related_terms`), **diagram (optional `diagram_id`)**, legal, choice traps, mistakes, memory tip, example.

## Internal links

- `related_terms`: only names that exist in the same CSV (表記ゆれ → `lookup_key`).
- Practice/ichimon CSV: add `term:用語名` in `tags` for question → term links.
- After changes: `python3 tools/build_all.py` must pass all validators.

## Public vs operator content

Never publish: 検索意図, 記事種別, update_policy, original_note, CSV/template how-to text in body columns.

## Build

```bash
python3 tools/validate_csv.py
python3 tools/audit_editorial_quality.py
python3 tools/audit_glossary_article_quality.py
python3 tools/build_all.py
```

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
