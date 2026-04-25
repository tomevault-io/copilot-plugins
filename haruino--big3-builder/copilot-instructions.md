## big3-builder

> 筋トレBIG3（ベンチプレス・スクワット・デッドリフト）のトレーニングプログラムを生成AIで生成し、記録更新を目指すWebアプリ。生成結果はGoogleスプレッドシートに出力・Drive保存する。現在開発初期段階。

# CLAUDE.md - Big3 Builder

## プロジェクト概要

筋トレBIG3（ベンチプレス・スクワット・デッドリフト）のトレーニングプログラムを生成AIで生成し、記録更新を目指すWebアプリ。生成結果はGoogleスプレッドシートに出力・Drive保存する。現在開発初期段階。

## 技術スタック

- **フレームワーク**: Next.js 16 (App Router) / React 19.2 / TypeScript
- **スタイリング**: Tailwind CSS v4 / shadcn/ui (class-variance-authority, clsx, tailwind-merge, lucide-react)
- **フォーム**: react-hook-form + zod(v4)バリデーション
- **ユーティリティ**: date-fns (日付操作)
- **データフェッチ**: SWR / fetch (axios不使用、fetch関数をラップして使用)
- **DB**: PostgreSQL (Supabase をホストとして利用、RLS不使用)
- **ORM**: Drizzle ORM (drizzle-kit でマイグレーション管理)
- **認証**: Auth.js v5 (next-auth beta, Googleログインプロバイダ)

## ディレクトリ構成

```
app/                  # Next.js App Router
  api/                # Route Handlers (API)
  view/
    (auth)/           # 未認証ユーザー用レイアウトグループ
    (authenticated)/  # 認証済みユーザー用レイアウトグループ
  test/               # テスト用ページ
components/           # 共通UIコンポーネント
docs/                 # プロジェクトドキュメント (要件定義、DBスキーマ設計など)
lib/
  db/
    drizzle.ts        # Drizzle クライアント初期化
    schema.ts         # Drizzle スキーマ定義
  supabase/
    client.ts         # Supabase クライアント (参考用、メインはDrizzle経由)
repositories/         # データアクセス層 (Repository パターン)
types/                # 型定義
```

## コマンド

```bash
npm run dev          # 開発サーバー起動
npm run build        # プロダクションビルド
npm run lint         # ESLint実行
npm run lint:fix     # ESLint自動修正
npm run format       # Prettier フォーマット
```

## コードスタイル・規約

- **Prettier設定**: セミコロンなし / ダブルクォート / インデント2スペース / printWidth 120 / trailingComma es5
- **ESLint**: next/core-web-vitals + next/typescript + prettier連携
  - `@typescript-eslint/no-explicit-any`: off
  - `react/no-children-prop`: off
  - `@next/next/no-async-client-component`: off
  - `@typescript-eslint/no-empty-object-type`: warn
  - `prefer-const`: off
  - `no-var`: error
  - `react-hooks/set-state-in-effect`: warn
- **パスエイリアス**: `@/*` でプロジェクトルートを参照 (例: `@/lib/db/drizzle`)
- **言語**: HTMLのlangは `ja`

## 開発ルール

- DBアクセスは Drizzle ORM 経由で行う（Supabase JSクライアントは直接使わない）
- 認証制御は Auth.js で行う（SupabaseのRLSは使わない）
- Repository パターンでデータアクセス層を分離する
- Route Handlers をAPIエンドポイントとして使用する
- App Router の Route Groups で認証状態別レイアウトを分ける: `(auth)` / `(authenticated)`

## Git運用ルール

### ブランチ戦略

- `main`: 本番ブランチ（直接コミット禁止）
- `develop`: 開発ブランチ（全PRのマージ先）
- 機能ブランチは **必ず `develop` から** 切る（mainから切らないこと）
- 各画面・機能ごとにブランチを作成する

### ブランチ命名規則

```
feature/○○    # 新機能追加（例: feature/login-page, feature/program-form）
fix/○○        # バグ修正（例: fix/auth-redirect）
refactor/○○   # リファクタリング（例: refactor/schema-cleanup）
```

### コミットメッセージ規則

プレフィックスを必ず付ける:

| プレフィックス | 用途 |
|------------|------|
| `feat:` | 新機能追加 |
| `fix:` | バグ修正 |
| `refactor:` | リファクタリング（機能変更なし） |
| `style:` | コードスタイル・フォーマット修正 |
| `chore:` | 設定変更・依存関係更新など雑務 |
| `docs:` | ドキュメント変更 |
| `test:` | テスト追加・修正 |

例: `feat: ログインページ追加`, `fix: 認証リダイレクトの修正`

### PR・マージルール

- PRは **必ず `develop` ブランチに向けて** 作成する（mainへの直接PRは禁止）
- `main` へのマージは `develop` からのみ行う
- PRにはやったことの概要を記載する

### 作業フロー

```
1. develop から feature/○○ ブランチを作成
2. 作業・コミット
3. develop に向けてPR作成
4. レビュー・マージ
5. リリース時に develop → main へマージ
```

## 環境変数 (.env)

- `DATABASE_URL` - Supabase PostgreSQL接続文字列 (SSL必須)
- `NEXT_PUBLIC_SUPABASE_URL` - Supabase プロジェクトURL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Supabase 匿名キー

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Haruino) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
