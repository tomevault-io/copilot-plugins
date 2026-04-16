## blog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 開発コマンド

### ビルドとテスト
- `pnpm dev` - 開発サーバー起動（Turbopack使用）
- `pnpm build` - 本番ビルド
- `pnpm start` - 本番サーバー起動
- `pnpm type-check` - TypeScriptの型チェック
- `pnpm test` または `pnpm test:run` - Vitestでテスト実行
- `pnpm check` - Biomeでコード検証
- `pnpm check:fix` - Biomeでコード修正

## プロジェクト構成

### アーキテクチャ
- **フレームワーク**: Next.js 15 (App Router)
- **CMS**: microCMS（ブログ記事とタグ管理）
- **デプロイ**: Vercel + GitHub Actions
- **スタイリング**: Tailwind CSS
- **テスト**: Vitest（メイン）、Jest設定もあり
- **パッケージマネージャー**: pnpm

### ディレクトリ構造
```
src/
├── app/           # Next.js App Router ページ
├── components/    # コンポーネント
│   ├── apps/      # アプリケーション固有コンポーネント
│   ├── features/  # 機能別コンポーネント
│   ├── ui/        # 汎用UIコンポーネント
│   └── layouts/   # レイアウトコンポーネント
├── libs/          # 外部サービス連携
├── types/         # TypeScript型定義
├── constants/     # 定数定義
└── utilities/     # ユーティリティ関数
```

### microCMS連携
- ブログ記事・タグデータは`src/libs/microcms.ts`で一元管理
- APIキーなどの環境変数は`next.config.ts`で設定
- 記事は`slug`ベースでルーティング

### テスト戦略
- テストファイルは対象ファイルと同じ階層に`xxx.test.ts`形式で配置
- Vitestで`src/**/*.test.{js,ts,jsx,tsx}`を実行
- `happy-dom`環境でReactコンポーネントテスト

### パスエイリアス
- `@/*` → `./src/*`
- `*` → `./src/types/*`

## コーディング規約

### 命名とスタイル
- TypeScript必須、Biome設定準拠
- 関数・変数名はキャメルケース
- コンポーネントはPascalCase
- ファイル名は対応するコンポーネント名と一致

### 禁止事項
- jQuery等レガシー技術の使用
- プロジェクト無関係な外部ライブラリ提案

### 必須作業
開発完了後は以下を実行してエラーがないことを確認：
1. `pnpm type-check`
2. `pnpm check`
3. `pnpm test`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dsk52) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
