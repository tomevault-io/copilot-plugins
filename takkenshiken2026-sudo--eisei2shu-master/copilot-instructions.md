## affiliate-banner

> アフィリエイト用バナー（試験ガイド一覧カード・記事内サムネ）の設定とCSS


# アフィリエイト用バナー

正本: `docs/affiliate/placement-and-rollout.md` §5・`affiliate-article-rules.md` §7・§8.4  
記事本文・CSV・brief・導線は `.cursor/rules/affiliate-article.mdc` を参照。

## 設置ページ（厳守）

| 設置する | 設置しない |
|----------|------------|
| `/articles/`・`/terms/`・`/q/` の **index のみ** | トップ・個別記事・個別用語・フッター |
| 各 `affiliate-*` 記事内（要点・hub） | 通常ガイド・用語詳細への画像バナー |

## バナーの種類

| 種別 | 置き場 | データ源 |
|------|--------|----------|
| **一覧カード（3ハブ共通）** | `articles/`・`terms/`・`q/` の各 `index.html` | `site-config.json` → `guideIndexPicks` |
| **要点右端サムネ** | 各アフィリエイト記事 | brief `products[0]` + `affiliate_product_ui.py` |
| **比較 hub カード** | 各アフィリエイト記事 | brief `products[]` + `affiliate_product_ui.py` |

記事内に独立ヒーローバナーは置かない（要点 + hub で代替）。

## guideIndexPicks（一覧バナー）

- **3枚固定**（講座 / テキスト / 問題集）。`href` は公開済み `affiliate-*` slug へ
- **`layout` 既定 `grid-3`**（運用標準）。`grid-2` は最大4件だが通常使わない
- **`leadsByHub`**（任意）: `articles` / `terms` / `q` でリード文を出し分け
- 各 item に **`image`** + **`imageAlt`** 必須。`images/affiliate/` の実ファイルを指す
- `kind`: `course` | `textbook` | `problem-book`（`mock` 可）。`kindLabel` は短く（例: 講座）
- 画像は brief と **同じファイルを再利用**可。講座=横長サムネ、書籍=表紙 webp
- 取得: `python3 tools/fetch_affiliate_product_images.py --slug <slug>`
- **`layout`**（任意・既定 `grid-3`）: `grid-3` | `grid-2` | `strip` | `compact` | `text`  
  - `grid-3`: 現行3列カード（一覧上部） / `grid-2`: 2列・最大2件 / `strip`: 横長帯 / `compact`: 画像小 / `text`: 画像なし
- HTML: `tools/guide_index_picks_ui.py` → `.hub-promo--{layout}`（`.article-index-pick-*` は後方互換）

## 画像ルール（共通）

- 自サイト `images/affiliate/*.webp` のみ（SVG プレースホルダー禁止）
- 命名: `{資格略称}-course-*` / `{資格略称}-book-*`（§7）
- 本番 CI は `build_all.py` で再生成するため、**画像ファイルもリポジトリにコミット**

## CSS（テンプレ正本）

| UI | ファイル | 主クラス |
|----|----------|----------|
| 一覧カード | `site-pages.css` | `.hub-promo-*` / `.article-index-pick-*` |
| 記事内カード・要点 | `seo-editorial.css` | `.affiliate-product-*`, `.seo-key-points-aside` |

一覧カード: **3枚等高・画像枠 2:1（max 148px）**・種別ラベルは右下・CTA は左下。  
`seo-editorial.css` 変更時は `tools/seo_editorial_chrome.py` の `SEO_EDITORIAL_CSS_VER` を更新。

## ビルド（退行禁止）

`tools/build_article_pages.py` から **削除しない**:

- `load_affiliate_brief` / `affiliate_product_hub_html` / `affiliate_key_points_box_html`
- `build_guide_index_picks_html` / `guide_index_picks()`

テンプレ同期後は必ず `python3 tools/build_article_pages.py` を実行し、アフィリエイト記事 HTML に  
`affiliate-product-hub` と `<img … images/affiliate/` が含まれることを確認。

## 反映手順

1. `exam-site-shell` で `site-config.json` / CSS / 画像を編集（**各サイトは独自設定**）
2. `python3 tools/build_article_pages.py` + `build_glossary_pages.py` + `build_past_question_pages.py`（または `build_all.py`）
3. 本番: `sync_from_template.py --target …` → サイト側で `build_all.py` → デプロイ

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
