## term-diagrams

> HTML diagrams embedded in glossary and question pages (JSON + diagram_id)


# Term Diagrams (HTML 図解)

**Full source of truth:** `docs/term-diagrams.md`

## Architecture

- **Embed in articles**, not standalone public diagram URLs.
- Data: `data/term_diagrams/{id}.json` → HTML via `tools/term_diagram.py`
- Reference: optional CSV column **`diagram_id`** (same ID, no extension)
- One JSON may be reused on **glossary term + question** pages

## Implemented vs planned

| Page | Status |
|------|--------|
| Glossary `terms/*.html` | ✅ After 「定義と基本理解」→ section 「図解で理解する」 |
| `q/past/`, `q/practice/`, `q/ichimon/` | 📋 Contract in docs; add `diagram_id` to CSV + insert in `q_explanation` when implementing |

## New site

1. Copy template; add JSON under `data/term_diagrams/` if needed
2. Add `diagram_id` column to `glossary_terms.csv`; set only rows that need diagrams
3. `python3 tools/build_all.py`

## Existing site

1. Fix **exam-site-shell** template only (unless user names production path)
2. `sync_from_template.py --target <path> --build` (manifest includes `term_diagram.py`, CSS, builders)
3. On production: add/edit JSON in `data/term_diagrams/`, set CSV `diagram_id`, `build_all.py`
4. Do **not** hand-edit generated `terms/` or `q/`

## Writing rules

- Opt-in: confusing **pairs** only (建ぺい率/容積率, 35条/37条, …)
- Body = definition; diagram = comparison + formula + numeric example; compare table = axis detail
- No duplicate copy across body / diagram / compare article

## Design (required — `docs/term-diagrams.md` §6)

- **Only** `.term-diagram-*` in `site-pages.css`; use CSS variables (`--ink`, `--green`, `--bg`, `--border2`, `--r`, `--r2`, `--sh`). No inline `style=`, no per-site diagram CSS forks, no raw hex in JSON/HTML.
- Tone: clean exam-site (white cards, thin borders, light shadow). Accent for examples/quiz = `--green`; pill labels = `--ink` background.
- Layout: `compare_dual` = 2 cards; **≥768px** two columns, mobile single column. Visuals `land` / `floors` = **14rem × 11rem**, 2px `--ink` lines, text labels only (`aria-hidden` on visuals).
- Changing look: edit template `site-pages.css` → check `terms/diagram-samples/kenpei-yoseki.html` → `sync_from_template.py` for existing sites. Update `term_diagram.py` if HTML structure changes.

## JSON (`compare_dual`)

`left` / `right`: `label`, `catch`, `formula`, `example`, `memo`, `visual` (`land` | `floors`)

## Validation

`validate_csv.py` → ERROR if `diagram_id` points to missing JSON.

## Samples (noindex unless noted)

- `terms/diagram-samples/` — diagram-only preview
- `terms/g-diagram-sample.html` — embedded writing sample (noindex)
- `terms/g-diagram-demo.html` — CSV pipeline demo (from `glossary_terms.csv`)

## When adding question-page support

Follow `docs/term-diagrams.md` §6: `diagram_id` on question CSVs; insert after 「正解の理由」 in `build_explanation_html`; reuse `diagram_body_html()`.

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
