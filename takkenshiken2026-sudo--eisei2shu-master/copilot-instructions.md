## editorial-quality

> Expert/pro-writer editorial quality for guide articles and glossary terms


# Editorial Quality (Expert × Pro Writer)

**Full guide:** `docs/editorial-quality.md`

Apply when writing or reviewing `data/guide_articles.csv`, `data/glossary_terms.csv`, scaffolds, or enrich scripts.

## Standard

Articles must read like **exam specialists + professional writers**: concrete, exam-linked, trustworthy—not template filler.

## Machine checks

| Tool | Role |
|------|------|
| `tools/editorial_quality.py` | Shared thresholds, placeholders, readability, generic phrases |
| `tools/guide_article_rules.py` | Guide: published rows need **180+ char sections**, **100+ char FAQ** |
| `tools/glossary_term_rules.py` | Terms: **180+ body**, **100+ FAQ×4**, 2+ paragraphs |
| `tools/audit_editorial_quality.py` | Full audit; `--strict` = WARN fails |
| `tools/build_all.py` | Runs `audit_editorial_quality.py` (ERROR gate) |

## Writing (always)

- です・ます; short sentences; explain jargon; who/when/what; summarize law, do not paste statutes
- No `【記入】`, `差し替えてください`, operator/CSV how-to in public columns
- FAQ answers must differ by role (definition / mistake / exam / next step for terms)

## Guide (`published`)

- `section_*_body`: **≥180 chars** each (with heading)
- `faq_*_answer`: **≥100 chars**
- `lead` ≥80, `related_links` 2+ internal slugs, trust columns filled, real official URLs

## Glossary (all rows)

- `term_detail_body`: **≥180 chars**, **≥2 paragraphs** (`\n\n`)
- `faq_1`–`faq_4` answers: **≥100 chars** each
- `explanation` ≥80; `related_terms` 2+ registered terms
- Practice CSV: `term:用語名` in `tags` when linked

## Before publish

```bash
python3 tools/validate_csv.py
python3 tools/audit_editorial_quality.py --strict
python3 tools/build_all.py
```

Do not lower thresholds to pass thin content—rewrite CSV instead.

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
