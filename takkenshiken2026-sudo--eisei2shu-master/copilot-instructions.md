## affiliate-article

> アフィリエイト記事の CSV・brief 編集時のルール（exam-site-shell 正本）


# アフィリエイト記事ルール

正本: `docs/affiliate/placement-and-rollout.md`（設置・導線）・`affiliate-article-rules.md` §3・§8

## 収益導線（最優先）

```
通常ガイド → affiliate-* 比較記事 → ASP
```

| 層 | ASP 直リンク |
|----|--------------|
| 通常ガイド（`affiliate-*` 以外） | **禁止** |
| 比較記事（`tags` にアフィリエイト） | **可**（hub・要点・本文） |

- 通常ガイド: `related_links` に比較記事 slug **1〜2 本** + 本文に slug **1文**
- フッター一括・用語全ページバナーはしない
- 設置箇所一覧: `placement-and-rollout.md` §2

## 執筆の必須原則（最優先・必ず運用）

### 価格は URL から最新を調査

- 公開・更新前に **各商品の ASP / 公式 URL を開き**、表示価格を確認してから brief・本文に反映
- テンプレ・他記事・推測からの転記禁止。brief `price_yen` と本文表記を一致させる
- 確認後 `fact_checked_at` を更新

### オリジナル執筆（機械的量産禁止）

- テンプレ・scaffold 出力は **構成の雛形のみ**。公開原稿としてそのまま使わない
- **1記事 = 1検索意図 = 1回の全面リライト**。タイトル・紹介商品・資格に沿った独自の section を書く
- 他 slug 流用・◯◯差し替えだけ・同一定型文の使い回し禁止
- 複数 slug の本文を一括生成して仕上げない

## 作成ゲート

- **ASP / 商品 URL 未確定 → CSV 行を作らない**（`scaffold --append` も不可）
- URL 確定後のみ `content_status=published` → ビルド → デプロイ
- リンク無し行は `draft` のまま（HTML 非生成）

## ページ構成（product-comparison）

1. 要点 → 目次 → 信頼性 → **section1 選び方** → **比較 hub** → 詳細 → FAQ → 関連6件
- **「この記事でわかること」section は書かない**（CSV に残っていてもビルドでスキップ）
- hub は **section 1 直後**（`affiliate_hub_after_section=1`）

## 要点ボックス

- intro: `user_intent`
- リスト: brief `products[].name` 上位3 + CSV `key_points`（`;` 区切り、最大2件推奨）→ **青リンク**（`affiliate_body_links.py`）
- 右下: 1位商品表紙/サムネ + **下に名称ラベル**（ASP 1リンク・**黒下線・最大2行**）

## 比較表・名称リンク

- 比較表の **商品名 / 講座名列** → ASP **黒テキストリンク**（`.affiliate-compare-name-link`）
- 要点リスト内の商品名リンク（青）とは別スタイル。混在しても CSS で分離する

## 表記・リンク

- 本文・リード・FAQ: 商品名・講座名はビルド時 **「」自動付与** → ASP リンク（`affiliate_body_links.py`）
- CSV 本文は括弧なしで書いてよい
- **【PR・広告】等は載せない** / `アフィリエイト` タグは公開表に出さない

## brief / CSV

| 種別 | `comparison_kind` | テンプレ |
|------|-------------------|----------|
| 書籍 | `books` | `affiliate-textbooks-recommend` |
| 講座 | `courses` | `affiliate-online-course-compare` |

- `layout: product-comparison` + brief → 比較表・カード・要点 UI
- `related_links`: **非アフィリ3 + アフィリ3**（計6件）。ASP URL は関連ボックスに出さない
- **通常ガイド**の `related_links`: 比較記事 **1〜2 本** + 他ガイド。ASP URL 行は書かない
- 書籍カード: 縦1列、meta は価格のみ

## CSS 変更時

`seo-editorial.css` を触ったら `tools/seo_editorial_chrome.py` の `SEO_EDITORIAL_CSS_VER` を更新。

## ビルド退行（厳守）

`build_article_pages.py` をテンプレ同期・マージするとき、次を **必ず残す**:

- `affiliate_product_hub_html` / `affiliate_key_points_box_html` / `load_affiliate_brief`
- `body_text_transform` / `is_affiliate_skip_section` / `affiliate_related_box_html`

欠落すると CI 再生成で **記事内の商品画像・比較 hub が消える**。一覧バナーは `.cursor/rules/affiliate-banner.mdc` 参照。

## 他サイト展開

1. `sync_from_template.py --target …` でエンジン同期
2. サイト固有: CSV・brief・`images/affiliate/`・`site-config.json` の `guideIndexPicks`
3. `build_all.py` → デプロイ。手順: `placement-and-rollout.md` §6

## 完了時

```bash
python3 tools/validate_csv.py
python3 tools/build_article_pages.py
```

---
> Source: [takkenshiken2026-sudo/eisei2shu-master](https://github.com/takkenshiken2026-sudo/eisei2shu-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
