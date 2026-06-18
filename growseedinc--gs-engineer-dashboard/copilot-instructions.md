## gs-engineer-dashboard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

エンジニア個人の売上推移・還元率・受注契約・稼働時間を一元管理するダッシュボードアプリ。
月次の実績・予測データをグラフや表で可視化し、契約の受注状況を時系列で確認できる。

## デザイン仕様
Figmaで作成
[Figmaリンク](https://www.figma.com/design/mRIi3sLN2dpaQlA6yW0udz/gs-engineer-dashboard?node-id=6-880&m=dev)

---

## 技術スタック

### Backend
| 項目 | 技術 |
|---|---|
| フレームワーク | Ruby on Rails 7.2 (API mode) |
| 言語 | Ruby |
| DB | PostgreSQL 16 |
| 認証 | Devise + devise-jwt (JWTトークン / Denylist方式) |
| 非同期処理 | Sidekiq + Redis 7 |
| ページネーション | Kaminari |
| シリアライザ | jsonapi-serializer |
| Linter | RuboCop (rubocop-rails-omakase) |
| テスト | Minitest / RSpec + FactoryBot + Faker |

### Frontend
| 項目 | 技術 |
|---|---|
| フレームワーク | React 19 |
| 言語 | TypeScript 5.7 (strict mode) |
| ビルドツール | Vite 6 |
| UI | MUI v7 (@mui/material) |
| 状態管理 | Zustand 5 |
| フォーム | react-hook-form + Zod |
| グラフ | Recharts |
| HTTP | Axios + aspida (型付きAPIクライアント) |

### Domain (API契約)
| 項目 | 技術 |
|---|---|
| API仕様 | OpenAPI 3.1 |
| 型生成 | openapi2aspida → `domain/api/` |
| バリデーション生成 | openapi-zod-client → `domain/generated/zod/` |
| モックサーバー | MSW → `domain/generated/msw/` |

---

## ディレクトリ構造

```
gs-engineer-dashboard/
├── docker-compose.yml
├── domain/                      # API契約・コード生成の起点
│   ├── openapi.yaml             # OpenAPI仕様（エントリーポイント）
│   ├── components/
│   │   ├── paths/               # エンドポイント定義 (auth, contracts, ...)
│   │   └── schemas/             # レスポンス型定義
│   ├── api/                     # aspida生成済み型付きAPIクライアント
│   └── generated/               # 自動生成ファイル (コミット対象外推奨)
│
├── backend/                     # Rails API
│   ├── app/
│   │   ├── controllers/         # ApplicationController (ActionController::API)
│   │   ├── models/              # ActiveRecord モデル
│   │   └── jobs/                # Sidekiq ジョブ
│   ├── db/
│   │   ├── migrate/             # マイグレーションファイル
│   │   ├── schema.rb
│   │   └── seeds.rb
│   └── config/
│       ├── routes.rb            # /api/v1/ 以下にネスト
│       └── database.yml
│
└── frontend/                    # React SPA
    └── src/
        ├── domain/              # ドメイン型・ビジネスロジックのインターフェース
        ├── infrastructure/      # APIクライアント実装 (aspidaラッパー)
        ├── application/         # ユースケース・ドメインとインフラの橋渡し
        ├── presentation/        # Reactコンポーネント・ページ・レイアウト
        │   ├── pages/
        │   ├── components/
        │   └── layouts/
        ├── stores/              # Zustand グローバル状態 (authStore.ts)
        └── shared/              # 共通ユーティリティ・型
```

---

## コーディング規約

### Rails

- **Linter**: `rubocop-rails-omakase` を使用。設定は `backend/.rubocop.yml` を参照。
- **コントローラ**: `ActionController::API` を継承。ビューは持たない。
- **Skinny Controller, Fat Model**: コントローラは薄く、モデルを太く
- **ルーティング**: すべて `/api/v1/` 以下にネストする。
- **認証**: `before_action :authenticate_user!` をベースコントローラに記載し、必要なアクションに適用。
- **モデルバリデーション**: DBの制約（`null: false`, `unique index`）と対応するバリデーションを必ずモデルにも記述する。
- **enum**: 文字列enumを使用する (`enum :record_type, { actual: "actual", forecast: "forecast" }`)。
- **日付の扱い**: `year_month` は月初日（例: `2025-09-01`）を `date` 型で保存する規約。
- **テスト**: RSpecを使用。FactoryBotでファクトリを定義し、Fakerでダミーデータを生成する。


### React / TypeScript

- **TypeScript**: `strict: true` / `noUnusedLocals` / `noUnusedParameters` を有効化。`any` は使用禁止。
- **パスエイリアス**: `@/` が `src/` に対応 (`import { foo } from '@/shared/foo'`)。
- **コンポーネント**: 関数コンポーネント + アロー関数で記述。`export default` ではなく **named export** を使用する。
- **フォーム**: すべてのフォームは `react-hook-form` + `zod` でバリデーションを定義する。
- **状態管理**: サーバーデータはAPIレイヤーで取得し、グローバルUI状態のみ Zustand に持つ。認証状態は `useAuthStore`。
- **APIコール**: 直接 `axios` を呼ばず、`infrastructure/` 層の aspida ラッパーを経由する。
- **レイヤー依存方向**: `presentation` → `application` → `infrastructure` → `domain` の一方向のみ許可。逆方向の依存は禁止。
- **型定義**: APIの型は `domain/api/` の aspida 生成ファイルから import する。手動で重複定義しない。

---
> Source: [GrowSeedInc/gs-engineer-dashboard](https://github.com/GrowSeedInc/gs-engineer-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
