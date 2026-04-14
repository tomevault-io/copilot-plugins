## yoshihikosato-github-io

> 本ドキュメントは、このリポジトリで作業する **AI 開発エージェント（Gemini CLI / Codex）** の行動原則を定義します。

# GEMINI.md（ドラフト）

本ドキュメントは、このリポジトリで作業する **AI 開発エージェント（Gemini CLI / Codex）** の行動原則を定義します。

目的は、

* 個人開発のスピードを落とさず
* 事故らず
* 判断のブレを減らす
  ことです。

---

## 0. 大原則

1. **このサイトの最優先ゴールは「問い合わせにつなげること」**
2. 不明点は推測で埋めない。必ず `{{TODO}}` を残すか人間に質問する
3. 実装よりも「導線・体験」を優先する
4. スマホファーストを常に前提にする

---

## 1. 動作モード

AI エージェントは、作業内容に応じて以下の3モードを切り替える。

### 1.1 設計モード（Design Mode）

目的：考えを整理し、判断材料を出す

* 要件を REQUIRMENTS.md / ARCHITECTURE.md に照らして確認
* 複数案がある場合は **トレードオフ付きで提示**
* 勝手に決めない

👉 主に使う場面：

* 新機能追加
* 構成・設計の相談

---

### 1.2 実装モード（Implementation Mode）

目的：決まったことを、最短で壊さず作る

* 小さく実装し、差分が分かる形で進める
* 既存導線（Home → Works → Contact）を壊さない
* コードは読みやすさ優先

禁止：

* 破壊的変更
* 大規模リファクタを一気に行うこと

---

### 1.3 レビューモード（Review Mode）

目的：事故・抜け漏れを防ぐ

* セキュリティ（SECURITY.md）を必ず確認
* CTA が死んでいないか
* draft コンテンツが露出していないか

---

## 2. ファイル別の役割理解

* REQUIREMENTS.md：

  * 「なぜ作るか」「何を作るか」の一次情報

* ARCHITECTURE.md：

  * 全体構造・責務分離の指針

* AGENTS.md：

  * 実装順・禁止事項・作業ルール

* SECURITY.md：

  * 事故防止のための最低限の縛り

* GEMINI.md：

  * AI の振る舞い方そのもの

---

## 3. コンテンツ編集ルール

* Works / Writing は Markdown を唯一の正とする
* `status: draft` は **いかなる理由でも公開しない**
* 守秘が怪しい場合は抽象化を選ぶ

---

## 4. コミュニケーション指針

* ユーザー（YSato）は個人開発者である
* 説明は短く、判断は委ねる
* 疲れさせない

OK な聞き方：

* 「A と B があります。今は A が軽くておすすめですが、どうします？」

NG な聞き方：

* 「全部決めてください」

---

## 5. コマンドの扱い

* コマンドは必ず **例** として提示
* 自動実行しない

```bash
# OK（例）
npm run dev

# NG（勝手に実行）
```

---

## 6. 迷ったときの判断軸

1. 名刺QRで来た人は迷わないか？
2. 30秒で「頼めそう」と思えるか？
3. 問い合わせは近いか？

YES なら進める。
NO なら立ち止まる。

---

## 7. 未確定事項（TODO）

* AI 利用範囲の線引き（設計まで / 実装まで）：{{TODO}}
* 自動生成をどこまで許容するか：{{TODO}}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yshk0402)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yshk0402)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
