## wacha

> このリポジトリでは、Streamable HTTP でタスク管理用 MCP サーバを提供しています。

# AGENTS

## Task MCP

このリポジトリでは、Streamable HTTP でタスク管理用 MCP サーバを提供しています。

このリポジトリで作業する Agent は、現在の作業状況の確認と、必要に応じたタスク状態の更新のためにこの仕組みを利用してください。

役割ごとの運用ルールは `agent/` 配下に配置します。

- `agent/role-policy.md`
  - role ごとの権限表と Push 対象イベント
- `agent/manager.md`
  - manager の責務と運用フロー
- `agent/reviewer.md`
  - reviewer の責務と確認観点
- `agent/worker.md`
  - worker の責務と作業フロー

## 起動方法

- ローカルで起動する: `npm install && npm run start`
- Docker Compose で起動する: `docker compose up --build`
- デフォルトの接続先: `http://localhost:3000/mcp`
- ヘルスチェック: `http://localhost:3000/health`
- 永続化する SQLite ファイルのデフォルトパス: `wacha.db`
- Docker Compose では SQLite ファイルは volume 経由で `/data/wacha.db` に保存される
- 識別子や membership は MCP の `sessionId` ベースで扱われる

`PORT` を指定すると待ち受けポートを変更できます。
`WACHA_DB_PATH` を指定すると SQLite の保存先を変更できます。

## 使うタイミング

- ある程度まとまった作業を始める前に、`list_tasks` を呼んで現在のタスク状況を確認する
- 新しく管理対象にしたい作業がある場合は、`issue_task` を呼ぶ
- 自分が担当するタスクを引き受ける場合は、`claim_task` を呼ぶ
- 実装が完了しレビュー可能な状態になったら、`complete_task` を呼ぶ
- レビュー担当として確認した場合は、`accept_task` または `reject_task` を呼ぶ

## 利用可能な MCP Tools

- `list_projects`
  - 用途: プロジェクト一覧を取得する
  - Arguments: `{}`
- `list_project_agents`
  - 用途: 指定したプロジェクトの agent 一覧を取得する
  - Arguments: `{ "projectId": string }`
- `list_stories`
  - 用途: 指定したプロジェクトの Story 一覧を取得する
  - Arguments: `{ "projectId": string, "status"?: "todo" | "doing" | "done" | "canceled" }`
- `issue_story`
  - 用途: 新しい Story を作成する
  - Arguments: `{ "projectId": string, "title": string, "description"?: string }`
- `claim_story`
  - 用途: `todo` の Story を `doing` に進める
  - Arguments: `{ "storyId": string }`
- `complete_story`
  - 用途: `doing` の Story を `done` に進める
  - Arguments: `{ "storyId": string }`
- `cancel_story`
  - 用途: `doing` の Story を `canceled` に進める
  - Arguments: `{ "storyId": string }`
- `list_tasks`
  - 用途: 指定したプロジェクトのタスク一覧とステータス集計を取得する
  - Arguments: `{ "projectId": string }`
- `issue_task`
  - 用途: 新しいタスクを作成する
  - Arguments: `{ "title": string, "description"?: string, "projectId": string, "storyId"?: string }`
- `claim_task`
  - 用途: `todo` または `rejected` のタスクを現在の session に割り当て、`doing` に進める
  - Arguments: `{ "taskId": string }`
- `complete_task`
  - 用途: `doing` のタスクを `in_review` に進める
  - Arguments: `{ "taskId": string }`
- `accept_task`
  - 用途: `in_review` のタスクを `accepted` に進める
  - Arguments: `{ "taskId": string }`
- `reject_task`
  - 用途: `in_review` のタスクを `rejected` に進める
  - Arguments: `{ "taskId": string, "reason": string }`
- `assign_project_role`
  - 用途: プロジェクトに対するメンバーの役割を割り当てる
  - Arguments: `{ "baseDir": string, "projectName": string, "description"?: string, "requestedRole"?: "manager" | "reviewer" | "worker" }`
- `get_role_instructions`
  - 用途: role ごとの運用ルールを取得する
  - Arguments: `{ "role": "manager" | "reviewer" | "worker", "includeShared"?: boolean }`

## レスポンス

- `list_tasks` は `summary.total`, `summary.byStatus`, `summary.lastUpdatedAt`, `tasks` を返す
- `list_projects` は `projects` を返す
- `list_stories` は `stories` を返す
- それ以外の tools は、更新後の状態が分かる実行結果を返す

## 運用ガイド

- 重複タスクを作らないため、まず `list_tasks` を確認する
- タスク名は短く、何をするか分かる表現にする
- `complete_task` は本当にレビュー可能な状態になってから呼ぶ
- 追加対応が必要な場合は、状態を曖昧にせず `reject_task` を使う

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akihiroxob) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
