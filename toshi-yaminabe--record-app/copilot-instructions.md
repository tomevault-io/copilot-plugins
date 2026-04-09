## record-app

> 音声録音 → 自動文字起こし → AI提案生成 を軸とした自己管理アプリ。

# record-app

音声録音 → 自動文字起こし → AI提案生成 を軸とした自己管理アプリ。
「分人（Bunjin）」という人格ファセット概念で、状況に応じた自己の使い分けを支援する。

**Tech Stack:** Next.js 15 (App Router) + Prisma 6 + Supabase PostgreSQL + Flutter (Android) + Gemini API
**Hosting:** Vercel (Serverless) / GitHub Releases (APK配布)

---

## ファイルマップ

```
record-app/
├── CLAUDE.md                 ← このファイル（セッション開始時に自動読み込み）
├── KNOWLEDGE.md              ← 詳細ナレッジ（DB設計、API仕様、フロー図、ER図）
├── ISSUES.md                 ← 課題管理（CRITICAL〜LOWの起票・完了追跡）
├── .env.example              ← 環境変数テンプレート
├── package.json              ← v2.0.0-beta.18, dependencies一覧
├── next.config.mjs           ← Next.js設定（ESM, bodySizeLimit: 10mb）
├── vercel.json               ← Vercel関数タイムアウト（transcribe/proposals: 60s）
├── jsconfig.json             ← パスエイリアス設定
├── .eslintrc.json            ← ESLint設定
│
├── prisma/
│   ├── schema.prisma         ← 14モデル定義（後述）
│   ├── seed.mjs              ← 初期データ（デフォルト分人5件 + ルールツリー）
│   └── migrations/           ← Prismaマイグレーション
│
├── lib/                      ← バックエンド共通ライブラリ
│   ├── prisma.js             ← DB接続（Supabase PostgreSQL via PgBouncer）
│   ├── gemini.js             ← Gemini API（STT + 提案生成, 環境変数 > DB保存キー の優先順位）
│   ├── crypto.js             ← AES-256-GCM暗号化（APIキーのDB保存用）
│   ├── constants.js          ← 全定数（MOCK_USER_ID, ステータス, デフォルト分人, BUNJIN_LIMITS, GEMINI_API_TIMEOUT_MS）
│   ├── errors.js             ← AppError/ValidationError/NotFoundError/ConflictError + errorResponse()
│   ├── validators.js         ← タスク状態遷移 + STT状態遷移マトリクス, ルールツリーバリデーション, 日付検証
│   └── services/             ← ビジネスロジック層（10サービス）
│       ├── bunjin-service.js
│       ├── memory-service.js
│       ├── proposal-service.js
│       ├── rule-tree-service.js
│       ├── segment-service.js
│       ├── session-service.js
│       ├── swls-service.js
│       ├── task-service.js
│       ├── transcribe-service.js  ← 文字起こしビジネスロジック（route.jsから抽出）
│       └── weekly-service.js
│
├── app/
│   ├── api/                  ← 20 APIエンドポイント（REST, Supabase Auth JWT認証）
│   │   ├── bunjins/          ← 分人 CRUD (GET/POST/PATCH/DELETE)
│   │   ├── cron/archive-tasks/ ← タスク自動アーカイブ (GET, CRON_SECRET認証)
│   │   ├── health/           ← ヘルスチェック (GET, 認証不要)
│   │   ├── memories/         ← メモリー CRUD (GET/POST/PATCH)
│   │   ├── proposals/        ← 提案 生成/確定/却下 (GET/POST/PATCH)
│   │   ├── rule-trees/       ← ルールツリー 編集/公開 (GET/PUT/POST publish)
│   │   ├── segments/         ← セグメント 一覧/更新 (GET/PATCH)
│   │   ├── sessions/         ← セッション 管理 (GET/POST/PATCH)
│   │   ├── settings/         ← ユーザー設定 (GET/PUT/DELETE) ← Gemini APIキーのDB保存
│   │   ├── swls/             ← SWLS回答 (GET/POST, upsert)
│   │   ├── tasks/            ← タスク CRUD (GET/POST/PATCH)
│   │   ├── transcribe/       ← 音声文字起こし (POST multipart/GET)
│   │   └── weekly-review/    ← 週次レビュー (GET/POST)
│   ├── contexts/
│   │   └── auth-context.js   ← AuthProvider + useAuth() (Supabase Auth)
│   ├── lib/
│   │   └── supabase-client.js ← クライアント側Supabaseシングルトン
│   ├── login/
│   │   └── page.js           ← ログイン/サインアップUI
│   ├── components/           ← 共通UIコンポーネント (header, tab-navigation, etc.)
│   ├── features/             ← 機能別ビュー (bunjins, daily, history, tasks, etc.)
│   ├── hooks/                ← React hooks (use-api, use-bunjins, use-proposals, use-tasks)
│   ├── providers.js          ← Client Componentラッパー (AuthProvider)
│   ├── page.js               ← ダッシュボード（SPA, タブ切り替え, 認証ガード付き）
│   ├── layout.js             ← ルートレイアウト (Providers統合)
│   └── error.js              ← エラーバウンダリ
│
├── flutter_app/              ← Flutter モバイルアプリ (Android)
│   └── lib/
│       ├── main.dart                          ← エントリポイント (ProviderScope + deviceId初期化)
│       ├── core/
│       │   ├── constants.dart                 ← AppConstants (mockUserId, bunjinLimits) + ApiConfig (baseUrl)
│       │   ├── errors.dart                    ← カスタムエラー
│       │   └── app_logger.dart                ← カテゴリ別ロガー (API/DB/REC/QUEUE/LIFE)
│       ├── data/
│       │   ├── models/                        ← DTOモデル (session/task/bunjin/bunjin_summary/proposal/segment)
│       │   ├── repositories/                  ← APIリポジトリ (session/task/bunjin/proposal)
│       │   └── local/                         ← SQLiteオフラインキュー (offline_queue_db, queue_entry)
│       ├── presentation/
│       │   ├── pages/                         ← 画面 (home/daily/tasks/settings)
│       │   ├── providers/                     ← Riverpod providers (recording/session/task/bunjin/proposal)
│       │   └── widgets/                       ← 共通ウィジェット (bunjin_chip, status_badge)
│       └── services/
│           ├── device/device_id_service.dart   ← デバイスID永続化 (SharedPreferences)
│           ├── recording/                      ← 録音サービス (foreground + background + permissions)
│           ├── transcribe/transcribe_service.dart ← 文字起こしAPI呼び出し (multipart)
│           └── offline/                        ← オフラインキュー + 接続監視 + リトライ
│
├── docs/                     ← 追加ドキュメント
├── future/                   ← 将来要件・計画
├── public/                   ← 静的アセット
└── test-results/             ← デバッグログ置き場 (.gitignore)
```

---

## DBモデル一覧（14モデル）

| モデル | テーブル | 用途 |
|--------|---------|------|
| Transcript | transcripts | 文字起こし結果（レガシー、後方互換維持） |
| Bunjin | bunjins | 分人（人格ファセット）、デフォルト5+カスタム3=最大8 |
| RuleTree | rule_trees | ルールツリー（条件分岐で分人を自動決定） |
| RuleTreeNode | rule_tree_nodes | ルールツリーノード（自己参照、深度制限10） |
| PublishedVersion | published_versions | ルールツリーのJSONスナップショット |
| Session | sessions | 録音セッション (ACTIVE / STOPPED) |
| Segment | segments | 音声セグメント（Transcript後継、STT状態管理付き） |
| Proposal | proposals | AI提案（日次サマリー / タスク提案） |
| Task | tasks | タスク (TODO/DOING/DONE/ARCHIVED) |
| WeeklyExecution | weekly_executions | 週次実行記録 |
| SwlsResponse | swls_responses | SWLS回答（主観的幸福度、日次1回） |
| Memory | memories | メモリー（学習記録、append-only） |
| UserSettings | user_settings | ユーザー設定（暗号化済みAPIキー保存） |
| AudioDeletionLog | audio_deletion_logs | 音声削除ログ（STT完了後の削除追跡） |

詳細なER図・リレーション・制約は **KNOWLEDGE.md** Section 2 参照。

---

## 環境変数

| 変数名 | 用途 | 必須 |
|--------|------|------|
| `DATABASE_URL` | Supabase PostgreSQL接続文字列 (Pooler) | 必須 |
| `GEMINI_API_KEY` | Gemini API (STT + 提案生成) | 必須 (※DB保存でも代替可) |
| `ENCRYPTION_KEY` | AES-256-GCM暗号化キー (32byte hex) | 任意 (未設定時DATABASE_URLハッシュ) |
| `CRON_SECRET` | Cronジョブ認証トークン | 任意 |
| `NEXT_PUBLIC_SUPABASE_URL` | クライアント側Supabase URL | 必須 (Web認証用) |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | クライアント側Supabase anon key | 必須 (Web認証用) |
| `DEV_AUTH_BYPASS` | 開発環境で認証バイパス (`true`/`false`) | 任意 |

Flutter側: `API_BASE_URL` をビルド時に `--dart-define` で注入。

**認証フロー**: Web UI は Supabase Auth (メール+パスワード) でログイン。JWT は `use-api.js` フックで全API呼び出しに自動付与。サーバー側 `lib/middleware.js` が JWT を検証し userId を取得。開発環境では `DEV_AUTH_BYPASS=true` でモックユーザーにフォールバック。

**レートリミット**: `lib/rate-limit.js` はUpstash Redis経由。Upstash未設定時は警告ログのみでスキップ（本番でもthrowしない）。

---

## ビルド・実行コマンド

```bash
# バックエンド
cd ~/apps/record-app
npm install
npx prisma generate
npx prisma migrate dev      # 初回のみ
node prisma/seed.mjs         # 初回のみ
npm run dev                  # http://localhost:3000

# Flutter
cd ~/apps/record-app/flutter_app
flutter pub get
flutter run --dart-define=API_BASE_URL=http://10.0.2.2:3000

# 本番APKビルド（Auth有効、env/prod.jsonで全パラメータ注入）
flutter build apk --release --dart-define-from-file=env/prod.json < /dev/null

# Vercelデプロイ
vercel deploy --prod
```

---

## ドキュメントガイド

| ファイル | 何を見るか |
|---------|-----------|
| **CLAUDE.md** (このファイル) | プロジェクト全体像、ファイルマップ、環境設定 |
| **KNOWLEDGE.md** | DB設計詳細、API仕様、データフロー図、外部サービス依存 |
| **ISSUES.md** | アクティブ課題のサマリー (完了したら行を削除) |
| `.env.example` | 環境変数テンプレート |
| `prisma/schema.prisma` | 正式なDBスキーマ定義（14モデル） |
| `lib/constants.js` | 全ステータス定数、デフォルト分人定義 |
| `lib/validators.js` | タスク状態遷移マトリクス、ルールツリー検証ロジック |

## 課題管理

- **真の管理**: [GitHub Issues](https://github.com/toshi-yaminabe/record-app/issues) — 詳細・対応履歴・コミットリンク全てここ
- **ローカル参照**: `ISSUES.md` — アクティブ課題のissue番号と1行概要のみ
- **対応時**: issue番号をコミットメッセージに含める (`fix #5: ...`)
- **完了時**: ISSUES.md から該当行を削除（GitHub側で自動close）

### 現在のブロッカー
なし（2026-02-19時点。#58-#62は新機能実装が必要な構造的課題）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/toshi-yaminabe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/toshi-yaminabe)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
