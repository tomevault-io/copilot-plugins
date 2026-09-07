## site-integration

> One-pass integration for q/ modes, footer nav, and terms index


# Site Integration

**正本:** `docs/integration-checklist.md`  
**機械検証:** `python3 tools/validate_site_integration.py`（`build_all.py` に含まれる）

## 期待値

- **テンプレ / フル同期サイト:** `site-config.json` 修正 → **`build_all.py` 1回**で静的 HTML + `index.html` フッターまで揃う（`apply_site_config` が SPA フッターを再生成）
- **宅建等:** フル同期不可。`sites/takken-master/SITE.md` のフェーズ同期 + 各フェーズで `build_all`

テンプレに **`FOOTER_ROOT_HREFS` は無い**。静的フッターは `footer_href()`（相対）、3モードタブは `q_hub_links_html()`（`/q/...` 絶対パス）。

## フッター「過去問一覧」

- **正本:** `site-config.json` → `navigation.footer` … `href: q/index.html`
- **index.html:** 手編集しない。`apply_site_config` → `update_index_shell_footer` が `site_shell_footer()` で同期
- フッターに「実践演習一覧」「一問一答一覧」を**勝手に追加しない**

## q/ ハブ・実践

- `build_q_index()` に `q_hub_links_html(rel_path, current="past")`
- `build_all.py` で `build_practice_ichimon_pages.py` を `build_past` の**直後**に実行
- 実践一覧の表行は `build_practice_index_table_row`（`#past-play-*` 付き past 用行を使わない）
- 実践・一問一答一覧は **`groupBy: category`** + `categoryOrder`（`site-config` の `fields` 順）
- 一問一答を **年度別グループに戻さない**
- 詳細・取り込み: **`docs/question-static-pages.md`** + **`.cursor/rules/practice-ichimon-static.mdc`**

## 用語一覧

- `terms_index_item_dict`: **`shortDef` と `definition`**
- `site-terms-index.js`: **`item.shortDef || item.definition`**

## 作業手順

1. 契約に沿ってソースを修正（生成 `q/` は触らない）
2. `python3 tools/build_all.py`
3. `validate_site_integration` / `validate_internal_links` が通ることを確認

## 本番

- パス明示がない限り本番リポジトリは編集しない
- 本番のフッターは**本番の `site-config.json` + 本番での `build_all`** が必要

## 禁止

| 禁止 | 理由 |
|------|------|
| 過去問一覧 → `q/practice/index.html` | 実践ページ上で自己リンク化 |
| index フッターだけ手直しして site-config 放置 | 次の `build_all` で上書き or 不整合 |
| 宅建へフル `sync_from_template` | `build_all` / `build_past` 破壊 |
| 生成 HTML に **手書き topnav** / **`q-static-header`** | フッター遷移でヘッダーが別物になる（[site-chrome.md](../docs/site-chrome.md)） |

## Site chrome（ヘッダー・フッター）

**正本:** [docs/site-chrome.md](../docs/site-chrome.md) / **Cursor:** [site-chrome-nav.mdc](./site-chrome-nav.mdc)

- ヘッダーは **`site_page_header()` のみ**。フッターは **`site_page_footer()`**
- **ヘッダー「過去問」= SPA 演習 `/#past`**。**フッター「過去問一覧」= 静的 `q/index.html`**（別コンテンツ・混同禁止）
- ヘッダー `current` active: `terms` / `practice` / `ichimon` のみ。**`q` は active にしない**
- フッター `current`: `site-config.json` → `navigation.footer[].key`（`q` = 過去問一覧）
- SPA ハッシュ（過去問・記録・復習）は **`/_learning_nav_href` → `/#past` 等のルート絶対パス**
- `html_footer.py` 変更後は **`patch_header_nav.py` または `build_all.py`**
- `validate_site_integration.py` が `tnav-past` / `q/index.html` の active 状態を検証

## Responsive（レスポンシブ）

**正本:** [docs/responsive-layout.md](../docs/responsive-layout.md) / **Cursor:** [responsive-layout.mdc](./responsive-layout.mdc)

- 静的ページは **`site-pages.css`**（「全ページ共通レスポンシブ」節必須）。旧 ~1.6k 行 CSS は **未同期**
- 全公開 HTML: **viewport** + `site-pages.css`（+ `site-theme.css`）
- サイト別 `*-mobile.css` **禁止**
- 375px 目視: `/`, `articles/index.html`, `terms/index.html`, `q/index.html`

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
