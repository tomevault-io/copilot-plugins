## seo-article-template

> SEO article and term-page structure for exam sites


# SEO Article Template

Use this rule when editing exam-site SEO articles, term pages, article CSVs, generators, or shared page CSS.

**Full source of truth:** `docs/seo-article-guidelines.md`  
## Editorial quality (mandatory)

- **Original prose:** Do not ship scaffold/template paragraphs copied across articles. Each `section_*_body` and FAQ must be unique to that article's search intent. See `docs/editorial-quality.md` §オリジナル執筆.
- `python3 tools/audit_editorial_quality.py` flags boilerplate phrases (`EDITORIAL_BOILERPLATE_PHRASES`) and cross-article duplicate section bodies as **ERROR** for published guides.
- Full source: `docs/editorial-quality.md` (`tools/audit_editorial_quality.py`, in `build_all`)

## Content volume

- Production: **100+ guide articles** (`data/guide_articles.csv`), **~10 affiliate articles** (`tags` must include `アフィリエイト`; under `articles/`, linked from other guides), **300+ glossary terms** (`data/glossary_terms.csv`) — **every term is a full detail article** (not optional columns). Genre whitelist: `site-config.json` → `guideArticleGenres` (12 labels). Affiliate rules: `docs/seo-article-guidelines.md` + `docs/affiliate/`. Slug examples: `docs/guide-article-catalog.md`. New guide row: `python3 tools/scaffold_guide_article.py` — see `docs/guide-article-template.md`. New term row: `python3 tools/scaffold_glossary_term.py` — see `docs/glossary-term-template.md`. Affiliate from theme: `python3 tools/scaffold_affiliate_article.py` — see `docs/affiliate/auto-create-workflow.md`.

## Article structure

- Order: lead → TOC → reliability → what-you-can-do → body sections → FAQ → article basic info → official info → related links.
- Reliability and basic info are **tables**. Reliability rows: author, reviewer/check, fact-check date, primary sources only.
- Body h2 uses small number badges; reliability, FAQ, basic info, official blocks are **not** numbered.
- Readable width ~860px desktop; mobile-friendly collapse.
- `Noto Sans JP`; font sizes from scale: 11, 12, 13, 14, 16, 19, 24px.
- Table headers: bg `#2b2b2b`, text white; body cells `#121212`.

## Public vs operator content (mandatory)

**Never show on public pages:**

- `update_policy`, `original_note` in any table
- Term pages: table rows 検索意図, 記事種別
- Articles index: sections like 「共通テンプレの増やし方」
- Body sections aimed at editors (e.g. 「記事を増やすときの例」) — keep in CSV notes, not published HTML

**OK on public tables:** reliability fields above; guide ジャンル/タグ; term 対象試験/分野/重要度/関連タグ/法令・根拠 when present.

Guide `primary_sources` from CSV; term pages fall back to `site-config.json` `externalLinks` so 主な参照元 is never empty.

## Accent color & article index UI

- Brand: `site-config.json` → `theme.accent` → `site-theme.css` tokens (`--accent-soft`, `--accent-emphasis`, hover tokens via `tools/site_config.py`).
- Body/links stay `#333333` (`--ink`); do not paint all text with accent.
- `<strong>` in articles: `--accent-emphasis`.
- Labels (meta-category, section numbers, field badges): soft pill (light bg + border), not solid dark fills (except `.site-page-mark`).
- **Article index cards:** neutral white cards only — **no genre-colored card backgrounds or top borders**.
- **Genre color only on labels:** `.article-index-card-genre` and filter chips (`site-pages.css` `--article-genre-*`).
- Hovers: subtle accent tint; avoid loud shadows or full-card genre tints.

## Internal links (zero broken hrefs)

- `related_links` slugs must exist in `guide_articles.csv` (CSV validation **errors** if missing).
- `related_terms` must be **2+ registered term names** in the same CSV (unregistered → **ERROR**, no `<a>` emitted).
- Generators: only output links to built pages; fallbacks use verified slugs/terms only.
- Term TOC: include anchors only for sections that are actually rendered.
- After changes: `python3 tools/build_all.py` must pass:
  - `validate_csv.py`
  - `validate_generated_seo.py`
  - `validate_internal_links.py` (all internal `href` + in-page `#anchors`)
- Do not ship if any link check fails. New SPA hashes on `index.html` require updating `INDEX_HASH_WHITELIST` in `validate_internal_links.py`.

## Term pages (glossary)

- **Every term** in `glossary_terms.csv` must be a full article — see `docs/glossary-term-template.md` and `.cursor/rules/glossary-term-template.mdc`.
- Required detail columns enforced by `tools/glossary_term_rules.py` / `validate_csv.py` (120+ char body, 2+ exam points, 2+ related terms, 2 FAQs, example Q/A).

## Build

- Run `python3 tools/build_all.py` after CSV, generator, config, or `site-pages.css` / `site-theme.css` changes.
- Keep generated HTML and `public_site/` in sync.

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
