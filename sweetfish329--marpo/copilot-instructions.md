## marpo

> Marp を使用してプレゼンテーションスライドをオンラインで共同編集するための Web アプリケーション。

# Marpo - Marp Online Collaboration Tool

Marp を使用してプレゼンテーションスライドをオンラインで共同編集するための Web アプリケーション。

## Bash Commands

```bash
# Backend
go run cmd/server/main.go         # サーバー起動 (port: 8080)

# Frontend (web/ ディレクトリで実行)
bun run start                     # 開発サーバー起動
bun run build                     # プロダクションビルド
bun run lint                      # Biome lint
bun run format                    # Biome format
```

## Architecture

### Backend (`internal/`)

| パッケージ | 説明 |
|-----------|------|
| `db/` | SQLite リポジトリ（ルーム管理、ドキュメント更新の永続化） |
| `websocket/` | WebSocket Hub/Room/Client パターンによる共同編集 |

#### WebSocket 設計

- **Hub**: アクティブなルームを一元管理（`sync.Mutex` でスレッドセーフ）
- **Room**: クライアント集約とメッセージブロードキャスト、クライアントゼロで自動クリーンアップ
- **Client**: Ping/Pong ハートビート（30秒間隔）で接続監視

### Frontend (`web/src/app/`)

| ディレクトリ | 説明 |
|------------|------|
| `editor/` | Monaco Editor + Yjs 統合コンポーネント |
| `services/` | FileService, CollaborationService |

## Core Files

- `cmd/server/main.go`: バックエンドエントリポイント（Echo フレームワーク）
- `web/src/app/app.ts`: Angular ルートコンポーネント
- `web/src/app/app.html`: メインテンプレート
- `web/src/app/app.scss`: コンポーネントスタイル
- `web/src/styles.scss`: グローバルスタイル
- `web/src/theme.scss`: Angular Material カスタムテーマ (Fuji-murasaki)

## Code Style

### TypeScript / Angular

- **IMPORTANT**: Signals (`input()`, `signal()`, `computed()`, `viewChild()`) を使用すること
- **IMPORTANT**: `ChangeDetectionStrategy.OnPush` を必須とする
- ES Modules (`import/export`) を使用、CommonJS は禁止
- 全てのコメント・ドキュメントは日本語 (UTF-8)

### Go

- Standard Go Project Layout に準拠
- `internal/` パッケージで外部からの直接参照を防止
- エラーハンドリング: `fmt.Errorf("context: %w", err)` でラップ

### SCSS

- CSS 変数はグローバルに `:root` で定義 (`--color-*`, `--radius-*`, `--shadow-*`)
- コンポーネント固有のスタイルは `app.scss` に記述
- Angular Material のオーバーライドは `styles.scss` に記述

## UI Design System

### Theme: Fuji-murasaki (藤紫)

- **Primary Color**: `#8a7cb8`
- **Default**: ダークモード (`data-theme="dark"`)
- **Switching**: `localStorage` + システム設定自動検出

### Components (IMPORTANT)

**基本的に Angular Material コンポーネントを使用すること。Material で UI/UX 要件を満たせない場合のみカスタム HTML や他ライブラリを検討する。**

#### Material コンポーネントのスタイルオーバーライド

`styles.scss` でグローバルにオーバーライドし、以下のデザイン要件を満たす：

- **Inputs (`mat-form-field`)**: 枠線非表示、角丸、ソリッド背景
- **Buttons (`mat-raised-button`)**: ピル型 (`border-radius: 9999px`)、グラデーション背景
- **Modal**: Glassmorphism (`backdrop-filter: blur(16px)`)、角丸28px
- **Theme Toggle**: `mat-slide-toggle` を使用

### Animations

- モーダル出現: scale + translateY のスプリングアニメーション (`--ease-out-back`)
- ボタン: hover でグロー効果 + 浮遊、active で scale(0.96)
- アイコンボタン: hover でパルスアニメーション
- 空状態: アイコンのフローティング、テキストのスタガードフェードイン
- ドロップダウン: スライドイン + オプションの横移動
- スピナー: 非線形回転 + グロー効果
- テーマ切替: 全要素に smooth transition

### Editor Features

- **Marp サジェスト**: `Ctrl+Space` でディレクティブ・特殊コメント・画像キーワードを補完
- **ページ区切り自動フォーマット**: `---` の上に自動で空行を追加
- **ルーム切り替え時のプレビュークリア**: 前のルームのコンテンツが残らないように

## API Endpoints

| Method | Endpoint | 説明 |
|--------|----------|------|
| GET | `/api/rooms` | ルーム一覧取得 |
| POST | `/api/rooms` | ルーム作成 |
| DELETE | `/api/rooms/:roomId` | ルーム削除 |
| POST | `/api/export` | スライドエクスポート (PDF/PPTX/PNG/MD) |

## Testing

- **IMPORTANT**: Antigravity Browser Control でUIの実動作を確認すること
- 開発サーバー起動後、`http://localhost:4200/` でテスト
- モーダル表示、入力、ボタン動作、テーマ切替を必ず検証
- テストが完了したら開発用に起動したサーバーを停止する。

## Tech Stack

| Layer    | Technology                                     |
| -------- | ---------------------------------------------- |
| Backend  | Go 1.24+, Echo, gorilla/websocket, SQLite      |
| Frontend | Angular 21, Bun, Biome, SCSS                   |
| Editor   | Monaco Editor                                  |
| Collab   | Yjs, y-websocket                               |
| Slides   | @marp-team/marp-core                           |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sweetfish329)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sweetfish329)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
