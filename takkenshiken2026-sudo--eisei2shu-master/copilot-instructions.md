## site-workflow

> Site integration, chrome nav, responsive layout, practice/ichimon CSV


# サイト統合・UI（要約）

**正本:** `docs/ORGANIZATION.md` → マスター `integration-checklist.md` + `site-chrome.md` + `responsive-layout.md` + `question-static-pages.md`

- フッター「過去問一覧」= `q/index.html`。ヘッダー「過去問」= SPA `/#past`（混同禁止）
- 生成 `q/`, `articles/`, `terms/` は手編集しない
- テンプレ修正 → `sync_from_template.py --target ~/Projects/<site>` → 本番 `build_all`

検証: `tools/validate_site_integration.py`（`build_all` 内）

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
