## agent-sdk-chat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

Agent SDK Chatは、Claude Agent SDKを使用したブラウザベースのチャットアプリケーションです。Next.js 15で構築され、ChatGPTライクなインターフェースでClaudeと対話でき、ファイル管理、ターミナルアクセス、カスタムモデル、ツール実行承認ワークフローなどの機能を備えています。

## よく使うコマンド

```bash
# 開発
npm run dev              # 開発サーバー起動（ホットリロード）
npm run build            # 本番ビルド（prisma generate + next build）
npm run start            # 本番サーバー起動

# コード品質
npm run lint             # ESLint実行
npm run typecheck        # TypeScript型チェック

# テスト
npm run test             # vitest（watchモード）
npm run test:run         # vitest（単発実行）
npm run test:coverage    # カバレッジ付きテスト実行

# データベース
npm run db:migrate       # Prismaマイグレーション（開発環境）
npm run db:deploy        # Prismaマイグレーション（本番環境）
npm run db:generate      # Prismaクライアント生成（スキーマ変更後に実行）
npm run db:studio        # Prisma Studio起動
```

## アーキテクチャ

### 技術スタック
- **フロントエンド**: React 19, Next.js 15 (App Router), Tailwind CSS 4, shadcn/ui
- **状態管理**: TanStack Query（サーバー状態）, React Context（UI状態）
- **バックエンド**: Next.js API Routes + Claude Agent SDK
- **データベース**: SQLite + Prisma ORM（出力先: `src/generated/prisma`）
- **ターミナル**: xterm.js（フロント）+ node-pty（バック）via WebSocket
- **国際化（i18n）**: next-intl（日本語/英語/中国語対応）
- **テスト**: Vitest + React Testing Library + jsdom

### 主要なデータフロー

**チャットストリーミング**:
1. ユーザーが`useChat()`フック経由でメッセージ送信
2. `POST /api/chat`がAPIキー認証後、SSE（Server-Sent Events）でレスポンスをストリーミング
   - `danger.useSubscriptionPlan`がfalseの場合: 設定されたAPIキーを使用（必須）
   - `danger.useSubscriptionPlan`がtrueの場合: サブスクリプションプランを使用（APIキー不要）
3. イベント種別: `text_delta`, `thinking_delta`, `tool_call`, `tool_result`, `tool_approval_request`
4. ツール実行は`canUseTool`コールバックと`ApprovalManager`でユーザー承認が必要
5. 承認タイムアウト: 設定画面で設定可能（デフォルト60分、0で無制限）、タイムアウト時は自動中断
6. 中断時は待機中の承認リクエストも中断され、途中結果がDBに保存される
7. ページリロード時: サーバーメモリから`isProcessing`と`pendingToolApproval`を復元し、ポーリングで処理完了を監視

**セッション管理**:
- セッションはClaude SDKの`claudeSessionId`を保持し会話を継続
- 各セッションは`settings`JSONフィールドに`workspacePath`を持つ
- メッセージはカーソルベースのページネーションで取得

### 主要モジュールの場所

| モジュール | 場所 | 役割 |
|-----------|------|------|
| チャットフック | `src/hooks/useChat.ts` | メッセージストリーミング、ツール承認、ページネーション、リロード時状態復元 |
| 承認マネージャー | `src/lib/approval-manager.ts` | Promiseベースのツール承認キュー、セッション紐づけ、タイムアウト・中断対応 |
| Claude SDK連携 | `src/lib/claude/` | SDK初期化、セッション管理 |
| ターミナルサーバー | `src/terminal-server/` | WebSocket PTYハンドラー |
| カスタムサーバー | `server.ts` | Next.js + WebSocket統合 |
| i18nプロバイダー | `src/i18n/` | クライアントサイドの国際化設定、ロケール検出 |
| i18nサーバー | `src/lib/i18n/` | サーバーサイドの翻訳関数 |
| 翻訳ファイル | `messages/` | 各言語のJSON翻訳ファイル（ja.json, en.json, zh.json） |
| テスト設定 | `vitest.config.ts`, `src/test/` | Vitest設定、セットアップ、モック |
| テストファイル | `src/**/__tests__/` | 各モジュールのユニットテスト |

### API構成

- **チャット**: `POST /api/chat`（SSE）, `/api/chat/approve`, `/api/chat/abort`
- **セッション**: `/api/sessions/[id]`でCRUD, `/api/sessions/[id]/messages`でメッセージ取得
- **モデル**: `/api/models`（標準）, `/api/models/custom`（カスタムモデル）
- **スキル**: `/api/skills`でCRUD（Claude Agent SDKスキル管理）
- **ワークスペース**: `/api/workspace/*`（一覧、クローン、ファイル操作）
- **ターミナル**: WebSocket `ws://localhost:3000/api/terminal`

### データベースモデル（Prisma）

- **Session**: `claudeSessionId`, `settings`（JSON）, `allowedTools`（JSON）, `skillSettings`（JSON）
- **Message**: `role`, `content`, `toolCalls`（JSON）, トークン使用量, `thinkingContent`
- **CustomModel**: `baseModel`, `systemPrompt`, アイコンカスタマイズ, `skillSettings`（JSON）
- **Skill**: スキル定義（`name`, `content`（SKILL.md）, `isEnabled`）
- **MCPServer**: MCPサーバー設定（stdio/sse/httpタイプ）
- **Settings**: アプリ設定のKey-Valueストア（`general`, `permissions`, `sandbox`, `appearance`, `titleGeneration`, `danger`）
  - `GeneralSettings`: `language`（表示言語: ja/en/zh）, `defaultModel`, `defaultPermissionMode`など
  - `SandboxSettings`: `workspacePath`（ワークスペースパス）, `claudeMdTemplate`（ワークスペースのCLAUDE.mdテンプレート）
  - `DangerSettings`: `showUsage`（使用量表示の有効化、デフォルトfalse）

## 開発時の注意事項

- SQLiteのJSONフィールドは文字列として保存され、アプリ層でパースされる
- クライアントコンポーネントは`"use client"`ディレクティブが必要
- UIコンポーネント追加: `npx shadcn@latest add <component>`
- 各ディレクトリに詳細な`CLAUDE.md`があるので参照すること

### 国際化（i18n）

- クライアントコンポーネントでは`useTranslations()`フックを使用
  ```tsx
  const t = useTranslations('settings');
  return <span>{t('title')}</span>;
  ```
- サーバーサイド（API Routes）では`createServerTranslator()`を使用
  ```ts
  const t = await createServerTranslator('errors');
  return { error: t('notFound') };
  ```
- 翻訳キーは`messages/ja.json`を基準に追加（en.json, zh.jsonも同時に更新）
- 新しい翻訳は適切なネームスペースに追加（例: `settings.appearance.title`）

### テスト

- テストファイルは対象モジュールと同じディレクトリの`__tests__/`に配置
- API Routeテストでは`vi.mock()`でPrismaとSDKをモック
- Hooksテストでは`@testing-library/react`の`renderHook`を使用
- fake timersを使用する場合は`afterEach`で`vi.useRealTimers()`を呼び出すこと
- GitHub Actions CIで自動テスト実行（push/PR時）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KarakuriAgent)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KarakuriAgent)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
