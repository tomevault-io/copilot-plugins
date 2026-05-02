## home-page

> - `app/` は Next.js App

# リポジトリガイドライン

## プロジェクト構成・モジュール構成

- `app/` は Next.js App
  Router のエントリポイント（`app/layout.tsx`、`app/page.tsx` など）を含みます。
- `app/globals.css` はグローバルスタイルと Tailwind の設定です。
- `public/`
  はサイトルートで配信される静的アセット（画像、アイコンなど）を置きます。
- `Plan.md` は SEO 実装チェックリストとデプロイ手順を管理します。

## ビルド・テスト・開発コマンド

- `npm run dev` はローカル開発サーバーを `http://localhost:3000` で起動します。
- `npm run build` は本番ビルドを生成します。
- `npm run start` はビルド成果物から本番サーバーを起動します。
- `npm run lint` は Next.js core web
  vitals と TypeScript ルールで ESLint を実行します。

## コーディングスタイル・命名規則

- 言語: TypeScript（React / Next.js App Router）。
- フォーマット:
  ESLint を必須とし、Prettier は Markdown やテキストが多いファイルで
  `proseWrap: "always"` を使用。
- 命名: React コンポーネントは `PascalCase`、変数/関数は
  `camelCase`、ファイル名は
  `kebab-case`（ただし Next.js の慣例に従うものは除く。例: `page.tsx`）。
- インポート: `@/*` エイリアスを使った絶対パスを優先。

## テストガイドライン

- 自動テストフレームワークは未設定。
- 追加する場合は、ランナーを明記し、スクリプト（例:
  `npm run test`）を追加し、使用例をこのファイルに記載。

## コミット・PR ガイドライン

- コミット履歴は説明的で文形式（日本語が多い）。要約は変更内容に具体的に（例:
  “SEO計画を更新”、 “Add sitemap generation”）。
- PR には以下を含める:
  - 変更概要と理由
  - UI 変更がある場合はスクリーンショット/録画
  - 関連する `Plan.md` タスクや issue へのリンク

## セキュリティ・設定の注意

- 環境ファイル（`.env*`、`.env.local`）は無視されます。秘密情報は `.env.local`
  に置き、必要なキーは文書化。
- 解析用 ID やシークレットは、明示的な承認がない限りコミットしない。

---
> Source: [pandalize/home-page](https://github.com/pandalize/home-page) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
