## crewvia

> カンバン駆動でタスクを管理し、Taskvia（WebUI）と連携して承認フローを実現する。

# Multi-Agent System - CLAUDE.md

公開前提のマルチエージェントシステム。
カンバン駆動でタスクを管理し、Taskvia（WebUI）と連携して承認フローを実現する。

## コンセプト

- **タスクが主役**。エージェントはカードをこなすワーカー
- **Star Trek非依存**。汎用的・ポータブルな設計
- **Taskvia連携**。WebUIでカンバン可視化・承認・ナレッジログ
- **公開前提**。tmux依存を排除し、誰でもセットアップできる

---

## ロール

- **Director**（1名）: タスク分解・Worker割り当て・進捗管理。詳細は `agents/director.md`
- **Worker**（複数）: タスク実行・改善提案。名前はスキルに紐づき歴代継承される。詳細は `agents/worker.md`
- スキルタグ一覧: `agents/director.md` §5（`qa` スキルは crewvia-qa skill で手順定義）
- 名前プール・カスタマイズ: `config/worker-names.yaml`
- 自律改善ルール: `config/autonomous-improvement.yaml`

---

## Taskvia連携

- リポジトリ: `tyz-works/taskvia` / WebUI: `taskvia.vercel.app`
- 承認チャネル（`CREWVIA_APPROVAL_CHANNEL`）: `taskvia`（デフォルト）/ `ntfy` / `both`
- 承認フロー実装: `hooks/pre-tool-use.sh`, `hooks/lib_approval_channel.sh`
- ナレッジログ投稿: `hooks/post-tool-use.sh`（type: `knowledge` / `improvement` / `work`）
- Verification Push: `scripts/taskvia-verification-sync.sh`（運用詳細は `knowledge/review.md`）

---

## ディレクトリ構成

```
/
  config/
    worker-names.yaml   名前プール・カスタマイズ設定
    crewvia.yaml        システム設定（承認チャネル・WIP制限等）
  hooks/
    pre-tool-use.sh     PreToolUse hook（Taskvia承認）
    post-tool-use.sh    PostToolUse hook（ログ投稿）
  agents/
    director.md         Directorのシステムプロンプト
    worker.md           Workerのシステムプロンプト
  scripts/
    start.sh            マルチエージェント起動スクリプト
    plan.sh             タスクプラン管理 CLI（per-task / multi-mission）
    taskvia-sync.sh     queue → Taskvia 同期
  queue/                プラン置き場（plan.sh が管理）
    state.yaml          active mission slug + default_mission
    missions/<slug>/
      mission.yaml      title / status / next_task_id
      tasks/tNNN.md     frontmatter + Description / Result
    archive/            完了 mission の退避先
  registry/
    workers.yaml        Worker のスキル・経験値
    heartbeats/         watchdog 監視用
  CLAUDE.md             このファイル
  README.md             公開向けセットアップガイド
```

---

## 環境変数

| 変数名 | 用途 |
|---|---|
| `CREWVIA_TASKVIA` | Taskvia 連携モード: `enabled` / `disabled` / `ask`（最優先。config・フラグより上） |
| `TASKVIA_URL` | Taskvia WebUIのURL |
| `TASKVIA_TOKEN` | Taskvia API認証トークン（`disabled` 時は不要） |
| `AGENT_NAME` | 起動時に設定されるエージェント名 |
| `TASK_TITLE` | 現在担当中のタスクタイトル |
| `TASK_ID` | 現在担当中のカードID |
| `CREWVIA_REPO_ROOT` | crewvia リポジトリの絶対パス。worktree 内からでも crewvia tools を参照できる |
| `CREWVIA_QUEUE` | キューディレクトリの絶対パス（通常 `$CREWVIA_REPO_ROOT/queue`） |
| `CREWVIA_MISSION_SLUG` | 担当中ミッションの slug（plan.sh pull 後に `.crewvia-env` 経由で設定） |
| `CREWVIA_TASK_ID` | 担当中タスクの ID（plan.sh pull 後に設定） |
| `CREWVIA_TASK_SLUG` | タスクタイトルを kebab-case 化した slug（worktree パス末尾に使用） |
| `CREWVIA_PROJECT` | Taskvia に送るプロジェクト識別子。デフォルト: `crewvia` |
| `CREWVIA_APPROVAL_CHANNEL` | 承認通知チャネル: `taskvia` / `ntfy` / `both`（config `approval_channel.mode` より優先） |
| `NTFY_URL` | ntfy サーバーの URL。`approval_channel.ntfy.url` より優先 |
| `NTFY_TOPIC` | ntfy 通知トピック名。**必須** — 空のまま運用すると通知が silent skip される |
| `NTFY_USER` | ntfy Basic 認証ユーザー名。`auth-default-access: deny-all` サーバーでは必須 |
| `NTFY_PASS` | ntfy Basic 認証パスワード |
| `APPROVAL_TOKEN_TTL_SECONDS` | ntfy ワンタイムトークンの有効期限（秒）。デフォルト: 900 |
| `CREWVIA_VERIFICATION_UI` | Taskvia 側の verification UI 表示制御。**Taskvia の Vercel env に設定** |

---

## 設計原則

1. **tmux非依存** - tmuxがなくても動く。tmuxはオプション
2. **Taskvia非依存** - Taskvia未接続でもスタンドアロンで動作可能
3. **名前はポジション** - 同スキルWorkerは同名前を引き継ぐ
4. **公開前提** - ドメイン固有設定を外に出し、設定ファイルで全カスタマイズ可能

---

## 今後の拡張余地

- [x] ~~Planner ロール~~ → `planning` スキルでプランレビュー Worker (Priya) として実装済み。crewvia-plan-review skill 参照。
- [ ] `plan.sh status` の JSON 出力サポート（WIP 計測の grep を置き換える）
- [ ] task frontmatter にブランチ名を持たせて Worker に伝達
- [ ] mission 間の優先度設定（現状は default_mission 優先のみ）

---
> Source: [tyz-works/Crewvia](https://github.com/tyz-works/Crewvia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
