## demo-fun-gile

> このリポジトリでは、**Slack のスラッシュコマンドから GitHub Issue / Project を操作し、タスク管理やスプリント運用を支援するボット**を実装します。

# CLAUDE.md

このリポジトリでは、**Slack のスラッシュコマンドから GitHub Issue / Project を操作し、タスク管理やスプリント運用を支援するボット**を実装します。
開発は主に **Claude Code** を使う想定です。



---

## 必ず守ってほしいこと
- コミットメッセージは日本語で書く
- .env.localは機密情報が含まれるため絶対に参照しない


## 全体像

### 目的

* Slack 上で `/task` `/burndown` `/review` `/plan` などのコマンドから GitHub のタスク状況を確認・可視化・振り返り・計画できるようにする。
* バーンダウンチャートなどの **画像** と、スプリントレビュー / プランニング用の **Markdown レポート**を自動生成する。
* 将来的には、プロジェクトごとのチャンネル運用・組織 GitHub との連携・Slack ワークスペース制限なども踏まえた運用を前提とする。

---

## アーキテクチャ概要

### 使用技術

* **Slack Bot**

  * スラッシュコマンド: `/task`, `/burndown`, `/review`, `/plan`（他も追加予定）
  * Slack App / Bolt (or Slack Web API) は Cloudflare Workers 側から呼び出し

* **バックエンド**

  * **Cloudflare Workers**
  * **Hono (TypeScript)** を使用した API ルーティング
  * 各スラッシュコマンド → Workers のエンドポイントに HTTP POST

* **GitHub 連携**

  * GitHub REST / GraphQL API (将来的に GitHub App or PAT)
  * 会社の GitHub 組織を前提としたリポジトリアクセス
  * Slack ユーザーと GitHub ユーザーの紐付け（設定ファイル or DB 想定）

* **チャート生成**

  * ロジック: Cloudflare Workers 内の TypeScript でバーンダウンデータを計算
  * レンダリング: **Chart.js** を利用した画像生成

    * 方針: Workers から外部の Chart.js 画像生成サービス（例: QuickChart 的なもの）にリクエストし、PNG を取得
    * Slack には画像をファイルとしてアップロード or 画像 URL を添付

* **ストレージ / 設定**

  * 初期は **環境変数 + 設定ファイル (JSON/YAML)** ベースでも可
  * 将来的には KV / D1 などの Cloudflare ストレージを検討

---

## レポジトリ構造(仮)

```
apps/
	slack-worker/         # Cloudflare Workers + Hono で Slack エンドポイント
packages/
	core/                 # GitHub CLI を叩くロジック / 共通ドメインロジック
	charts/               # Chart.js でバーンダウン画像を作るロジック
scripts/
	fetch-issues.ts       # 単体実行で GitHub CLI からデータ取る検証スクリプトなど
.env.local              # ローカル用環境変数（Slack/GitHub のトークンなど）
wrangler.toml
package.json
pnpm-workspace.yaml
```

---

## スラッシュコマンド仕様（v0 設計）

### 1. `/task`

**目的:**
自分にアサインされている Issue を、Slack 上で素早く一覧取得する。

**デフォルト挙動:**

* 対象:

  * 「現在のイテレーション」に含まれる Issue
  * ステータスが `Todo` または `In Progress`
* フィルタ:

  * Slack コマンドを実行したユーザーに紐づく GitHub アカウントに `assignee` が設定されている Issue

**想定オプション例:**

* `--iteration=<name or number>`
  → 対象イテレーションを変更
* `--include-done` / `--status=done,todo,in-progress`
  → 完了済み Issue も含める
* `--repo=xxx/yyy`
  → リポジトリを限定（未指定時はチャンネルに紐づくデフォルトリポジトリ）

**レスポンス例 (Slack メッセージ):**

* 簡易なテーブル/ブロック形式

  * タイトル
  * ステータス
  * ストーリーポイント / 見積もり
  * GitHub Issue リンク

---

### 2. `/burndown`

**目的:**
指定期間（デフォルトは現在のイテレーション）のバーンダウンチャートを画像で返す。

**デフォルト挙動:**

* 対象:

  * 「今のイテレーション」全体
* 出力:

  * バーンダウンチャート画像（PNG）
  * テキストとして:

    * 総ポイント数
    * 残りポイント数
    * 進捗率（例: 完了ポイント / 総ポイント）
    * 予定ラインと実績ラインの簡易説明

**追加フィルタオプション例:**

* `--range=all`
  → 全期間（全イテレーション）のバーンダウン
* `--assignee=me`（デフォルト） or `--assignee=@user`
  → 特定メンバーのみのバーンダウン
* `--label=<label>` / `--category=<category>`
  → ラベルやカテゴリ別にポイント集計
* `--repo=xxx/yyy`
  → 対象リポジトリを指定

**チャート生成フロー:**

1. Workers + Hono で `/api/burndown` エンドポイントを実装
2. GitHub Projects / Issues からイテレーションの開始日・終了日・各 Issue の完了日・ポイントを取得
3. 日ごとの残ポイント配列を計算
4. Chart.js コンフィグを生成
5. 外部のチャート画像生成サービスにコンフィグを渡し、PNG を取得
6. Slack ファイルアップロード API で画像を貼り付けつつ、数値情報もメッセージに添付

---

### 3. `/review`

**目的:**
スプリント / イテレーションのレビューを **対話的に進め、最終的に Markdown レポートを生成** するコマンド。

**大まかなフロー:**

1. コマンド実行 → 「今イテレーションのバーンダウンチャート」をまず表示
2. ユーザーに「中間レビュー」か「スプリントレビュー」かを選ばせる（オプションで指定も可）
3. 種類に応じて聞く内容を変更（例）:

   * 中間レビュー:

     * 現在の進捗感
     * リスク / ブロッカー
     * 優先度変更の要否
   * スプリントレビュー:

     * 達成したこと
     * 達成できなかったこと
     * 良かった点 / 改善点
     * 次スプリントへのアクション
4. 対話ログ + バーンダウン画像 + 集計表をまとめて **Markdown ファイル** を生成
5. 生成された Markdown:

   * Slack にその場で貼る
   * or GitHub の特定リポジトリに `docs/reviews/yyyymmdd-<iteration>.md` として PR 化、など拡張余地あり

**想定オプション例:**

* `--type=mid` / `--type=sprint`
  → レビュー種別を直接指定
* `--iteration=<name>`
  → 対象イテレーションを指定

---

### 4. `/plan`

**目的:**
スプリントプランニングを支援し、取り組む予定のタスク一覧などを含む **Markdown 計画書** を作成する。

**大まかなフロー:**

1. `/plan` 実行 → 前イテレーションの結果や現在のバーンダウンを表示（任意）
2. 次スプリントの目標を対話的に決める
3. GitHub から候補 Issue を取得（例: `Todo` かつ未着手のバックログ）
4. 対話を通じて「今回スプリントでやる / やらない」を選別
5. 最終的に:

   * スプリントゴール
   * 取り組む Issue 一覧（タイトル / ポイント / 担当者）
   * リスク・懸念
   * 備考
     を Markdown としてまとめる
6. `/review` 同様、Slack に出力 or GitHub に保存など

**想定オプション例:**

* `--iteration=<name>`
  → 次スプリントのイテレーション名
* `--capacity=<point>`
  → チームのキャパシティを指定し、ポイント合計がオーバーしたら警告

---

## プロジェクトおよびチャンネルとの紐付け

* 会社の GitHub は **組織** で管理されており、プロジェクトごとに対象リポジトリが存在する。
* Slack では **プロジェクトごとにチャンネル** を作成しているため、

  * 「このチャンネルからのコマンドは、この GitHub リポジトリ / プロジェクトを対象にする」
  * というマッピングを設定ファイル or ストレージで管理する。
* ボットは **会社の Slack ワークスペース参加者のみ利用可能** にする。

  * Slack App 側のインストールスコープ・ワークスペース制限で制御

---

## Claude Code の使い方ガイド

Claude Code で開発する前提でのメモです。

### 開発タスクの粒度

Claude に投げるときは、以下のような単位でプロンプトすると扱いやすいです:

* 「`/task` の Hono ルートと GitHub API コール部分を実装して」
* 「バーンダウン計算用の純関数を TypeScript で実装してテストコードも書いて」
* 「この Chart.js のコンフィグ生成ロジックをリファクタして」
* 「Slack の Block Kit メッセージを、一覧表示しやすい形に整形して」

### 期待するコードスタイル

* 言語: **TypeScript**
* フレームワーク: **Hono**（Cloudflare Workers モード）
* Lint/Format:

  * `eslint` + `prettier` を利用
* 設計:

  * `routes/` に各スラッシュコマンド用 API を置く
  * `services/github.ts` などで GitHub 連携を抽象化
  * `services/chart.ts` でバーンダウンデータ → Chart.js コンフィグを生成
  * `services/slack.ts` で Slack のレスポンス生成

### Claude への依頼テンプレート例

```text
CLAUDE.md に従って、
- /api/burndown の Hono ルート
- GitHub から今イテレーションの Issue を取得し、日ごとの残ポイント配列を計算するロジック
を実装してください。

前提:
- Cloudflare Workers + Hono
- TypeScript
- chart.js の画像生成は外部サービスに投げる前提なので、ここではコンフィグ生成まででOK
```

### テスト方針

* **テストランナー**: **Vitest** を使用
* **Hono / Workers のテスト**:
  * `app.request()` を使ったシンプルなテスト
  * 実際の HTTP リクエスト/レスポンスをシミュレート
* **外部 API モック (Slack / GitHub)**:
  * **MSW (Mock Service Worker)** または **nock** で HTTP モック
  * 実際の API 呼び出しを避け、テストの高速化と安定化
* **ドメインロジックのテスト**:
  * バーンダウン計算などのロジックは **純粋関数** として実装
  * **ユニットテスト** に寄せることで、テストしやすく保守性を向上

---

## ローカル開発

※ ここはリポジトリ構成が固まり次第、細かく更新する想定。

### セットアップ例

```bash
# 依存インストール
npm install

# dev サーバ (miniflare 等)
npm run dev
```

### 必要な環境変数（例）

* `GITHUB_TOKEN` or `GITHUB_APP_PRIVATE_KEY`
* `SLACK_BOT_TOKEN`
* `SLACK_SIGNING_SECRET`
* `DEFAULT_GITHUB_ORG`
* チャンネル ↔ リポジトリのマッピングは、最初は `config/channel-map.json` などでもよい

---

## 今後の拡張アイデア（メモ）

* `/task` から直接「完了にする」「担当を変更する」などのアクションボタン実装
* `/burndown` で、期間・粒度（日/週）をインタラクティブに切り替え
* `/review` / `/plan` の Markdown を自動で GitHub Issue / Discussion / Docs に保存
* ユーザーごとに「レビュー質問セット」をカスタマイズできるようにする

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kkoba0925)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kkoba0925)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
