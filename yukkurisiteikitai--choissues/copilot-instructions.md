## choissues

> **「迷う時間をゼロにし、50分単位のリズムで確実に実行する」**

# FlowSync システム仕様書

**「迷う時間をゼロにし、50分単位のリズムで確実に実行する」**

---

## 📋 目次

1. [コンセプト](#コンセプト)
2. [システム構成](#システム構成)
3. [データ設計](#データ設計)
4. [各コンポーネントの機能](#各コンポーネントの機能)
5. [Toggl Track連携](#toggl-track連携)
6. [ユーザーフロー](#ユーザーフロー)
7. [技術スタック](#技術スタック)
8. [開発ロードマップ](#開発ロードマップ)

---

## コンセプト

FlowSyncは、GitHub Issues、Google Calendar、Discord、VS Code、Toggl Trackを統合し、**PDCAサイクルを自動化**するタスク管理システムです。

### システムの役割分担

- **脳 (Brain):** GitHub Issues - 全てのタスクの源泉
- **心臓 (Heart):** Google Calendar + 推論エンジン - 空き時間を計算し、タスクを割り振る
- **司令塔 (Commander):** Discord - 通知、開始/終了のトリガー
- **現場 (Cockpit):** VS Code - 今の作業への集中、タイマー表示
- **記録係 (Logger):** Toggl Track - 自動時間記録、実績データ収集 🆕

---

## システム構成

```
┌─────────────────┐
│ GitHub Issues   │ ← タスクのマスターデータ
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ Backend Logic   │ ← 空き時間計算 + タスク選定
│ (API Server)    │
└────┬───────┬────┘
     │       │
     ↓       ↓
┌─────────┐ ┌──────────────┐
│ Discord │ │ VS Code Ext  │
│  Bot    │ │              │
└─────────┘ └──────┬───────┘
                   │
                   ↓
            ┌──────────────┐
            │ Toggl Track  │ ← 実績記録
            └──────────────┘
```

---

## データ設計

### GitHub Issues ラベル運用ルール

すべてのIssueには以下のラベルを必須とします。

#### サイズラベル (必須)

- `size:S` : 15分未満（3〜4個で1スロット消化）
- `size:M` : 50分（1スロット消費の標準単位）
- `size:L` : 2スロット以上（システム対象外。分割しないとアサインされない）

#### 優先度ラベル

- `priority:high` : 優先的に選出
- `priority:normal` : 空きがあれば選出

### Toggl Track データマッピング

GitHub IssuesからToggl Trackへのデータ変換ルール：

| GitHub | Toggl Track | 例 |
|--------|-------------|-----|
| Issue番号 + タイトル | Description | `[#102] DBスキーマ設計` |
| リポジトリ名 | Project | `webapp-frontend` |
| サイズラベル | Tags | `Size:M`, `Size:S` |
| Issue種別 | Tags | `Feature`, `Bugfix` |
| - | Client | 案件名や組織名（任意） |

---

## 各コンポーネントの機能

### A. バックエンド (Logic / Server)

すべてのハブとなるAPIサーバー（Cloudflare Workers / Render等）

#### 1. 朝のバッチ処理 (Daily Plan Generator)

**Input:**
- Google Calendarの今日の予定（9:00〜16:00の間にある会議など）
- GitHub Issues (`status:open`, `assignee:me`)

**Process:**
- 可処分時間を計算（例: 会議除くと4時間空き＝約4スロット）
- 優先度順にIssueをピックアップ（例: Mサイズ3個 + Sサイズ詰め合わせ1個）

**Output:**
- 今日の「プレイリスト」を確定し、Discordへ投稿

#### 2. 状態同期 (State Sync)

- 「現在実行中のタスク」と「残り時間」を管理
- VS CodeとDiscordにリアルタイムでブロードキャスト

### B. Discord Bot (Commander)

スマホやサブモニタで操作するリモコン

#### 機能一覧

1. **朝の通知**
   ```
   🤖 FlowSync: 今日の空き時間は 4スロット です。
   1. [M] ログイン画面の実装 (#101)
   2. [M] DBスキーマ設計 (#102)
   3. [S-Pack] メール返信 / ライブラリ更新
   4. [M] リファクタリング (#105)
   
   [承認して開始] [再生成]
   ```

2. **実行管理**
   - コマンド: `/start [task_id]` (手動開始)
   - 通知: 50分経過時にメンション「休憩してください（5分）」

3. **完了報告**
   - ボタン: [完了(Close Issue)] / [中断(Next Slot)] / [Sサイズ1個完了]

4. **デイリーレポート（夕方）**
   ```
   🤖 Daily Report:
   今日は予定より +40分 オーバーしました。
   特に [#105] バグ修正 はMサイズ(50分)でしたが、実際は 90分 かかりました。
   次回からこの類のタスクは Lサイズ にするか、分割を検討してください。
   ```

### C. VS Code 拡張機能 (Cockpit)

コードを書いている最中にブラウザやDiscordを見に行かなくて済むUI

#### 1. ステータスバー表示

```
🔴 Rec [M] DB設計 (34:20)
```

- `🔴 Rec`: Togglが裏で正常に動いていることを示すインジケーター
- `[M]`: タスクサイズ
- `DB設計`: タスク名（短縮版）
- `(34:20)`: 残り時間カウントダウン

**エラー表示:**
- APIエラーでTogglが動いていない場合は灰色で警告

#### 2. サイドバー / コマンドパレット

- 今日のToDoリストを表示
- ワンクリックで「作業開始」「完了」を送信
- コマンド: `FlowSync: Start Next Task`

#### 3. Focus Mode（オプション）

- タスク開始時に自動で「禅モード（Zen Mode）」
- 現在担当しているIssueのDescriptionをWebViewで表示

---

## Toggl Track連携

### 動作トリガー（自動化）

ユーザーはTogglのアプリやWebを開く必要なし。**VS Codeの操作に連動**。

#### 1. タスク開始時 (`/start` or VS Codeボタン)

```javascript
// System → Toggl API を自動実行
POST /workspaces/{workspace_id}/time_entries
{
  "description": "[#102] DBスキーマ設計",
  "project_id": 12345, // リポジトリ名から検索
  "tags": ["Size:M", "Feature"],
  "created_with": "FlowSync"
}
```

#### 2. タスク中断・休憩時 (`/break` or 50分経過)

```javascript
// System → Toggl API を自動実行
PATCH /workspaces/{workspace_id}/time_entries/{id}/stop
```

#### 3. タスク完了時 (`/done`)

1. Togglをストップ
2. GitHub IssueをClose
3. (オプション) 実際にかかった時間をGitHub Issueのコメントに自動投稿
   ```
   Work time: 48min (estimated: 50min)
   ```

### 必要なAPI

**Toggl Track API v9:**

- `GET /me/projects` - リポジトリ名に対応するプロジェクトIDを取得
- `POST /workspaces/{workspace_id}/time_entries` - タイマー開始
- `PATCH /workspaces/{workspace_id}/time_entries/{time_entry_id}/stop` - タイマー停止

**トークン管理:**
- 環境変数 `TOGGL_API_TOKEN` にAPIトークンを設定

### 見積もり精度の向上

**シナリオ例:**

1. **朝:** システムが「今日の空き時間は4時間なので、Mサイズ(50分)タスクを4つアサインします」と提案
2. **実行:**
   - 1つ目のタスク: 45分で完了（Toggl記録: 45min）→ 優秀！
   - 2つ目のタスク: 苦戦して90分経過（Toggl記録: 1h 30min）→ アラート発生
3. **フィードバック（夕方のDiscord通知）:**
   - 予定との差分を可視化
   - 未来のタスク選定アルゴリズムの精度向上に活用

---

## ユーザーフロー

### 09:00 - プランニング (Discord)

Discordに通知が来る：

```
🤖 FlowSync: 今日の空き時間は 4スロット です。

1. [M] ログイン画面の実装 (#101)
2. [M] DBスキーマ設計 (#102)
3. [S-Pack] メール返信 / ライブラリ更新 / ドキュメント修正
4. [M] リファクタリング (#105)

[Start Plan] [再生成]
```

あなたは **[Start Plan]** ボタンを押す。

### 09:05 - 第1スロット開始 (VS Code)

- VS Codeを開くと、ステータスバーに `🔴 Rec [M] ログイン画面の実装 (49:59)` が表示
- 自動的にタイマーがスタート
- **Toggl Trackが裏で自動起動**
- 作業に集中する

### 09:55 - 休憩通知 (Discord/VS Code)

- VS Codeの右下にトースト通知: `Time's up! Take a break.`
- Discordに通知: `🍵 50分経過しました。休憩してください。[完了] [延長]`
- **Togglが自動停止**
- あなたはコーヒーを淹れに行く

### 10:05 - 次のタスクへ

- 休憩から戻り、VS Codeのコマンドパレットで `FlowSync: Start Next Task` を実行
- Issue #101 は自動でCloseされる
- Toggl記録が自動保存される
- 次は `[M] DBスキーマ設計` がセットされ、新しいTogglエントリーが開始

### 17:00 - デイリーレポート (Discord)

```
🤖 Daily Report:
✅ 完了: 3タスク
⏱️ 実績: 4時間12分 (予定: 4時間)
📊 見積もり精度: 85%

詳細:
- [#101] ログイン画面: 48分 (予定50分) ✅
- [#102] DBスキーマ: 52分 (予定50分) ✅
- [#105] リファクタリング: 90分 (予定50分) ⚠️

💡 [#105]のようなタスクは次回Lサイズで登録することをお勧めします。
```

---

## 技術スタック

### バックエンド

- **言語:** TypeScript
- **フレームワーク:** Hono or Express
- **ホスティング:** Cloudflare Workers / Render / Vercel
- **データベース:** Supabase or Firestore（タスク進行状態の一時保存）

### 連携サービス

- **Discord:** discord.js
- **VS Code:** VS Code Extension API (TypeScript)
- **GitHub:** Octokit (GitHub API)
- **Google Calendar:** Google Calendar API
- **Toggl Track:** Toggl Track API v9

---

## 開発ロードマップ

### Phase 1: Basic Sync（基本連携）

**目標:** Discord通知だけでも価値を出す

- [ ] GitHub IssuesからタスクをFetch
- [ ] Google Calendarから空き時間を計算
- [ ] タスク選定アルゴリズム実装
- [ ] Discordに朝の通知を送信

**成果:** 「何やるんだっけ？」が消える

### Phase 2: Execution & Tracking（実行と記録）🔥

**目標:** VS Code拡張 + Toggl連携の完成（最重要フェーズ）

- [ ] VS Code拡張機能の開発
  - [ ] ステータスバー表示
  - [ ] サイドバーのタスクリスト
  - [ ] コマンドパレット統合
- [ ] Toggl Track API連携
  - [ ] タスク開始時の自動タイマー起動
  - [ ] タスク完了時の自動停止
  - [ ] 記録漏れの検知と警告
- [ ] GitHub Issueの自動Close
- [ ] Discord通知との双方向同期

**成果:** 完全自動記録が実現。見積もり精度が上がり始める

### Phase 3: Scheduling Logic（スケジューリング最適化）

**目標:** AIによる賢いタスク選定

- [ ] カレンダー連携の高度化
- [ ] 過去の実績データを元にした見積もり補正
- [ ] タスクの自動分割提案
- [ ] デイリーレポートの生成

**成果:** システムが勝手に最適なプランを組んでくれる

### Phase 4: Advanced Features（拡張機能）

- [ ] モバイルアプリ（React Native）
- [ ] チーム機能（複数人での利用）
- [ ] ダッシュボード（データ可視化）
- [ ] Slack連携
- [ ] Notion連携

---

## 補足情報

### 環境変数

```bash
# GitHub
GITHUB_TOKEN=ghp_xxxxx
GITHUB_USERNAME=your-username

# Google Calendar
GOOGLE_CLIENT_ID=xxxxx
GOOGLE_CLIENT_SECRET=xxxxx
GOOGLE_REFRESH_TOKEN=xxxxx

# Discord
DISCORD_BOT_TOKEN=xxxxx
DISCORD_CHANNEL_ID=xxxxx

# Toggl Track
TOGGL_API_TOKEN=xxxxx
TOGGL_WORKSPACE_ID=xxxxx
```

### プロジェクトの価値

このシステムは**「フリーランスやエンジニアの自己管理ツール」として製品化できるレベル**の設計です。

- タスク管理の自動化
- 実績ベースの見積もり精度向上
- PDCA サイクルの完全自動化
- 集中力の最大化（50分ルール）

---

## ライセンス

MIT License

---

## 貢献

プルリクエストを歓迎します。大きな変更の場合は、まずissueを開いて変更内容を議論してください。

---

**FlowSync - Flow state synchronized with reality.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yukkurisiteikitai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Yukkurisiteikitai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
