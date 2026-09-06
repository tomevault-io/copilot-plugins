## exam-site-shell-template

> Treat exam-site-shell as a template project


# exam-site-shell Template

- `~/Projects/exam-site-shell` is the template folder for exam site projects.
- Treat changes here as template-level changes that can affect generated or copied sites.
- When editing generated/public files, keep source/template files in sync when both exist.
- **Do not edit production site repos** (e.g. remote GitHub sites) unless the user explicitly names a path and asks to. Default: change template only; user syncs with `tools/sync_from_template.py --target <local path>`.
- UI/CSS/generators: edit here first; then `sync_from_template.py` + `build_all.py` on the production clone. Do not hand-edit generated `q/`, `articles/`, `terms/` on production.

## Documentation & rules

- **Start here:** `docs/ORGANIZATION.md` — folder roles, master docs, daily workflow.
- **Index:** `docs/README.md` — full doc list.
- **Cursor (2 files):** `.cursor/rules/site-workflow.mdc` (UI・q・sync), `.cursor/rules/content-authoring.mdc` (CSV・記事・用語). 有料模試は `.cursor/rules/paid-mock-exam.mdc` → `docs/paid-mock-exam.md`. 詳細は常に `docs/` 正本。
- **Multi-site:** `docs/multi-site-workflow.md` — `sync_from_template.py --target ~/Projects/<site>`.
- **This file:** template-wide defaults and build requirements.

## Content targets (production)

- **50以内** guide articles (`data/guide_articles.csv`); genres in `guideArticleGenres`, catalog in `docs/guide-article-catalog.md`
- **~10** affiliate guide articles (`tags` includes `アフィリエイト`; same `articles/` URLs; A8.net / afb / Amazon) — see `docs/affiliate/placement-and-rollout.md` and `docs/seo-article-guidelines.md`
- **300+** glossary terms (`data/glossary_terms.csv`); **every row is a full detail article** (see `docs/glossary-term-template.md`); `validate_csv.py` enforces minimum column quality
- Knowledge hub tabs (compare/numbers/mistakes) are **retired**; old URLs redirect via `build_hub_retire_redirects.py`.

## Reader-facing quality (non-negotiable)

1. **No broken internal links** — `build_all.py` runs `validate_internal_links.py`; fix before deploy.
2. **No operator content on public pages** — no 更新方針/独自メモ/検索意図/記事種別 in tables; no template-how-to sections on indexes; no editor-only body sections in published articles.
3. **Tables for trust & meta** — 執筆, 確認, 事実確認日, 主な参照元 on every SEO article/term page.
4. **Accent styling** — `theme.accent` via CSS tokens; labels colored, article index **cards** neutral; genre color on **badges/chips only** (`site-pages.css`).

## Typography & tables

- `#121212` headings/body in tables; `#333333` normal text/links; `#2b2b2b` table header cells with white text.
- Footer/static pages: do not use overly light gray for primary links.

## Build pipeline (required after CSV/generator/CSS/config changes)

```bash
python3 tools/build_all.py
```

Runs in order: `validate_csv.py` → generators → `validate_generated_seo.py` → `validate_site_integration.py` → `validate_internal_links.py` → `public_site/` sync.

Optional audits: `validate_csv.py` alone, `audit_article_freshness.py`.

## Key tools

| Tool | Role |
|------|------|
| `tools/site_config.py` | `site-theme.css` from `site-config.json` (accent tokens) |
| `tools/build_article_pages.py` | Guide articles + index |
| `tools/build_glossary_pages.py` | Term pages, field hubs, related links |
| `tools/validate_csv.py` | CSV + `related_links` slug / `related_terms` checks |
| `tools/validate_generated_seo.py` | Section order, forbidden table rows |
| `tools/validate_site_integration.py` | フッター・q タブ・用語・**実践/一問一答件数・分野順** |
| `tools/validate_internal_links.py` | Post-build href integrity |
| `tools/import_orig_questions_to_practice_csv.py` / `import_base_questions_to_ichimon_csv.py` | SPA バンク → CSV |
| `tools/scaffold_glossary_term.py` / `tools/glossary_term_rules.py` | 用語詳細記事 CSV 雛形・必須列ルール |
| `tools/enrich_o4_glossary_details.py` / `audit_glossary_article_quality.py` | 演習 DB からの用語充填・品質監査 |
| `tools/sync_from_template.py` | Copy shared engine to `--target` (required) |
| `tools/check_template_drift.py` | Diff template vs production before sync |

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
