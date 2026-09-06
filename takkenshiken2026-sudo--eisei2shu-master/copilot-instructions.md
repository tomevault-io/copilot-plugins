## site-chrome-nav

> Header/footer nav — past exercise vs past index list (site chrome)


# Site Chrome Nav（ヘッダー vs フッター）

**正本:** [docs/site-chrome.md](../docs/site-chrome.md)

## 別コンテンツ（混同禁止）

| UI | 行き先 | 役割 |
|----|--------|------|
| ヘッダー「過去問」 | `/#past` | SPA **演習** |
| フッター「過去問一覧」 | `q/index.html` | 静的 **一覧** |

## 実装契約（`html_footer.py`）

- `LEARNING_NAV_ITEMS`: `("tnav-past", "過去問", "#past", …)`
- `LEARNING_NAV_ACTIVE_BY_PAGE`: **`q` を含めない**（`terms` / `practice` / `ichimon` のみ）
- `_learning_nav_href`: `#` 始まり → **`"/" + dest`**（`/#past`, `/#dash`, `/#review`）
- `FOOTER_SUPPRESS_CURRENT_WHEN_HEADER`: 上記 3 キーのみ（`q` 禁止）
- `build_practice_*`: `current="practice"`（`current="q"` 禁止）

## 変更後の必須手順

```bash
python3 tools/patch_header_nav.py   # または build_all.py
python3 tools/validate_site_integration.py
```

## 禁止

- ヘッダー `tnav-past` → `q/index.html` または `../../q/index.html`
- `q/index.html` でヘッダー「過去問」に `active` / `aria-current`
- 生成 HTML への手書き topnav
- SPA ハッシュの相対パスだけ（試験ガイド深階層で壊れる）

## 目視 1 件

試験ガイド → ヘッダー「過去問」→ **`takken-master.jp/#past`**（一覧 `/q/` にならないこと）

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
