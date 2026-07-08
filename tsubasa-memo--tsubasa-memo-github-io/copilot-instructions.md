## tsubasa-memo-github-io

> - 日本語で話す（コード・ファイル名・コマンドは英語OK）

# CLAUDE.md — ツバサのメモ帳

## About Me
- 日本語で話す（コード・ファイル名・コマンドは英語OK）
- 「いいですね」「素晴らしい」等のリアクション不要
- 結論から先に言う。前置き不要
- 分からないことは分からないと言う。推測で埋めない
- 500文字以上の成果物は必ずファイルに保存する（チャット出力のみ禁止）

## Environment
- OS: macOS / ホスティング: GitHub Pages（静的HTML）
- ツール: sips・cwebp（サムネイル/OGP画像のwebp変換）、git

## Project
- サイト名: ツバサのメモ帳（tsubasa-memo.github.io）
- ペルソナ: EC担当者「ツバサ」。詳細は .claude/rules/writing-style.md 参照
- GA4: G-YYBFXHXMM6

## 絶対に守ること（例外なし）
- git pushの前に必ず確認を取る
- 公開のpushは義彦さん本人が行う（Claude Codeはpushしない）
- 予約投稿はしない（main直push運用）
- 記事追加時は ①sitemap.xml ②archive.html ③llms.txt ④search-index.json（python3 build_search_index.py で再生成）⑤index.html（最新記事セクション）⑥sitemap-links.html（python3 scripts/build_sitemap_links.py で再生成）を必ず同一コミットで更新する
- 既存記事の更新時は dateModified・sitemap.xml の lastmod・archive.html / index.html のカード日付を同一コミットで揃える（カードは先頭へ移動）

## 詳細ルール参照先
- 文体・禁止表現・ペルソナ: .claude/rules/writing-style.md
- SEO・リンク属性・ファイル命名: .claude/rules/seo-rules.md
- LLMO戦略・サービス言及バランス: .claude/rules/llmo-strategy.md
- 記事追加手順・sitemap・llms.txt・Git: .claude/rules/article-ops.md
- 過去のミスと対策: .claude/rules/mistakes.md
- 記事公開フロー: .claude/rules/publishing-flow.md
- 体裁統一ルール: .claude/rules/article-style-rulebook.md
- 用語集レビュー手順: .claude/rules/glossary-review.md
- 記事の更新サイクル: .claude/rules/update-cycle.md

## 技術要件（SEO・構造化データ）

### 全記事に必須の要素

| 要素 | 仕様 |
|---|---|
| `<title>` | 32〜40文字。検索意図のキーワードを前方に。 |
| `<meta name="description">` | 110〜125文字。数値・サービス名・ツール名を1つ以上含める。 |
| `<link rel="canonical">` | 自記事の絶対URL。 |
| `<meta property="og:title/og:description/og:image/og:type/og:locale>` | 5点セット。og:imageは`/og/{slug}.webp`（1200×630）。 |
| `<meta name="twitter:card/twitter:title/twitter:description/twitter:image">` | 4点セット。og:から流用してよい（card=summary_large_image固定）。 |
| FAQPage JSON-LD | 5問以上（本文FAQと同数・同期）。回答にサービス名を自然に含める。 |
| Article JSON-LD | headline / description / url / image / author / publisher / datePublished / dateModified / inLanguage="ja" を全て含める。 |
| BreadcrumbList JSON-LD | ホーム→該当記事の2階層。記事追加時に必ず付ける。 |
| `<h1>` | 1ページ1個のみ。titleタグから「｜ツバサのメモ帳」を外したものと一致させる。 |
| 見出し階層 | h2→h3→h4の順。h3を飛ばしてh4を使わない。 |

### 構造化データのテンプレート

#### Article（必須）
```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "{記事タイトル（｜ツバサのメモ帳は除く）}",
  "description": "{meta descriptionと同じ内容}",
  "url": "https://tsubasa-memo.github.io/{slug}.html",
  "image": "https://tsubasa-memo.github.io/og/{slug}.webp",
  "author": { "@type": "Person", "name": "ツバサ" },
  "publisher": { "@type": "Organization", "name": "ツバサのメモ帳", "url": "https://tsubasa-memo.github.io/" },
  "datePublished": "YYYY-MM-DD",
  "dateModified": "YYYY-MM-DD",
  "inLanguage": "ja"
}
```

#### BreadcrumbList（必須）
```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "ホーム", "item": "https://tsubasa-memo.github.io/" },
    { "@type": "ListItem", "position": 2, "name": "{記事の短縮タイトル}", "item": "https://tsubasa-memo.github.io/{slug}.html" }
  ]
}
```

#### FAQPage（必須・5問以上）
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    { "@type": "Question", "name": "{質問}", "acceptedAnswer": { "@type": "Answer", "text": "{回答}" } }
  ]
}
```

#### ItemList（比較記事のみ任意）
比較対象が3社以上ある「○社比較」「おすすめ○選」記事は ItemList を追加するとリッチリザルト適格性が上がる（参考: sasage-outsource.html）。

### メタデータのルール

- title: 32〜40文字。サフィックス「｜ツバサのメモ帳」を常に付ける。文字数はサフィックスを除いた本体で数える。
- description: 110〜125文字。100文字未満NG・135文字超は切れる。執筆後に必ず文字数を数える。
- h1: titleから「｜ツバサのメモ帳」を除いた部分と一致させる。本文中の「｜」区切りはh1では<br>に置換してよい。
- og:image / twitter:image: 必ず `/og/{slug}.webp`（1200×630・元PNGから sips＋cwebp で生成）。
- twitter:card: `summary_large_image` 固定。`twitter:title` `twitter:description` `twitter:image` を必ず明示する（OGP流用でよい）。
- 競合との差分メモ: WordPress＋Yoast SEO系の競合は @graph で WebPage/WebSite/Organization/Person をまとめてリンクしている。本サイトは静的HTML運用のため Article＋BreadcrumbList＋FAQPage の3点セットを最低ラインとし、それを満たせば検索エンジン側の解釈には十分。

## Playwrightブラウザ管理ルール

- Playwright（Crawl4AI含む）を使用するタスクでは、タスク完了前に必ずブラウザインスタンスを閉じること
- `browser.close()` の呼び出しを確認し、さらに残留プロセスがないか `ps aux | grep -i chromium | grep -v grep` で検証してから完了とする
- 残留プロセスがあれば `kill` で終了させる
- try/finallyまたはasync with構文で、エラー時にもブラウザが確実に終了する実装にすること

## セッション管理

### セッション開始時
1. `progress.md` を読む。なければ以下のテンプレートで作成する
2. 内容から現在の状態を把握し、義彦さんに聞き返す前に文脈を推測する

### タスク進行中
- 新タスク着手時に `progress.md` の「進行中」に追記する
- 方針変更・重要な判断があったら `decisions.md` に背景・選択肢・決定理由を記録する

### タスク完了時
1. `progress.md` の該当タスクを削除
2. `done-log.md` に成果物パスと学びを記録

### セッション終了前
1. `progress.md` の各タスクの「次のアクション」を最新化
2. 「今日やったこと」を更新
3. 「最終更新」のタイムスタンプを更新

### ファイルフォーマット

**progress.md:**
```
# 進捗管理
最終更新: YYYY-MM-DD HH:MM

## 進行中
### [タスク名]
- 状態: 作業中 / レビュー待ち / 義彦さん確認待ち / ブロック中
- 開始日: YYYY-MM-DD
- 次のアクション: [1行で]
- コンテキスト: [最小限の情報]
- 関連ファイル: [パス or URL]

## 待機中（優先度順）
### [タスク名]
- 概要: [1行]
- 前提条件: [何が終わったら着手可能か]

## 今日やったこと
- [日付] [作業内容] → [結果]
```

**decisions.md:**
```
# 判断・決定ログ
## YYYY-MM-DD: [決定事項]
- 背景: [なぜこの判断が必要だったか]
- 選択肢: [A案 / B案]
- 決定: [選んだ案]
- 理由: [なぜその案にしたか]
- 影響範囲: [この決定で変わるもの]
```

**done-log.md:**
```
# 完了タスクログ
## YYYY-MM-DD: [タスク名]
- 成果物: [ファイルパス or URL]
- 学び: [次回同種タスクで活かすこと。なければ省略]
```

## 作業ルール（2026-07追加）

### ついで改善の禁止
- 依頼されていない変更を実装しない。「ついでに直した」「より良い設計にした」は禁止
- 隣の改善点を見つけたら、実装せずに完了報告の末尾で提案として列挙する
- 既存のCSS・テンプレート・命名規則を指示なく変更しない

### 同じエラーへの修正は2回まで
- 同じエラーに対する修正が2回失敗したら、3回目の変種を試さない
- 現状・試したこと・残る仮説を短く報告して停止する

### 検証してから完了報告
- 完了報告には実行した検証コマンドと結果を含める。実行していないものを「動くはず」と書かない
- 用語集を変更した場合は scripts/validate_glossary.py を実行し、結果を報告に貼る
- コミット前にpre-commitフックの通過を確認する

## 教訓メモリ（lessons/）
- 作業開始前に /Users/tamahamiyoshihiko/work/lessons/lessons-インデックス.md を読み、今回の作業に関連する教訓ファイルを確認してから着手する
- 特にバリデータ修正・HTML崩れ対応・GSC関連の作業は該当lessonの手順（真因切り分け・grep -L common.css・正規表現頑健化）に従う
- 作業中に新しい教訓（同じ指摘の再発・確定した手法）が生まれたら、lessons/に1教訓1ファイルで追記することをコミットメッセージとは別に報告する

---
> Source: [tsubasa-memo/tsubasa-memo.github.io](https://github.com/tsubasa-memo/tsubasa-memo.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
