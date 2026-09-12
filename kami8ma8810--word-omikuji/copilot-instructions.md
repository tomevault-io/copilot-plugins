## word-omikuji

> - frontend/ は React + Vite のフロント。backend/ は Hono バックエンド。pnpm workspace で管理されています。

# Repository Guidelines

## Project Structure & Module Organization
- frontend/ は React + Vite のフロント。backend/ は Hono バックエンド。pnpm workspace で管理されています。
- docs/ に計画書や語彙資料を置き、data/processed/<language>.json に語彙データを配置します。生成スクリプトと checksum を必ず添付してください。
- 新規モジュール追加時は development-guidelines.md のクリーンアーキテクチャ区分と pnpm workspace 構成に従い、issue でディレクトリ計画を共有します。

**注記**: 将来的に apps/web、apps/api、packages/* 構成への移行を検討中です。現状は frontend/、backend/ のシンプルな構成を採用しています。

## Build, Test, and Development Commands
- `pnpm install` で依存同期。**Node.js 20.19+ または 22.12+** と pnpm 8 を前提に進めてください（Vite 7.x の要件）。
- Node.jsのバージョン管理には `.nvmrc` を使用（`nvm use` で自動切り替え）。
- フロントとバックを並列起動: `pnpm dev`
- 個別起動: `pnpm --filter frontend dev` または `pnpm --filter backend dev`
- DB が必要なら `docker compose up db` を先行実行します。
- 品質確認: `pnpm lint`、`pnpm test`、`pnpm typecheck`
- リリース確認: `pnpm build`

**注記**: Playwright E2Eテストは実装予定です。現状は Vitest + React Testing Library でコンポーネントテストを実施しています。

## Coding Style & Naming Conventions
- TypeScript は 2 スペースインデント・セミコロン必須・シングルクォート基準。`pnpm lint --fix` で整形します。
- React コンポーネントは PascalCase、hooks は useXxx、ユースケースは VerbNounUseCase、リポジトリは XxxRepository。ファイル名は kebab-case.ts(x)、テストは .test.ts /.spec.ts。
- Tailwind クラスは layout → spacing → typography の順序。shadcn/ui 上書きは意図をコメントで明記します。

## Testing Guidelines
- t-wada TDD を徹底し Red → Green → Refactor を記録。ユースケースは Vitest で分岐網羅、API は Hono test client、UI は React Testing Library。
- テストファイルは `*.test.ts` または `*.test.tsx` の命名規則で `__tests__/` ディレクトリに配置します。
- 現在152個のテストが実装済み（ユースケース層、リポジトリ層、カスタムフック、UIコンポーネント、ページコンポーネント）
- 語彙 JSON 更新時はスナップショットと差分検証を更新し、CI 前に `pnpm test` と `pnpm lint` の成功を確認します。

**注記**: Playwright E2Eテストとaxe-core a11yテストは今後実装予定です。

## Commit & Pull Request Guidelines
- Conventional Commits を厳守し 1 コミット 1 論点。テスト緑化後に commit してください。
- PR には背景、変更概要、テストログ、必要に応じてスクリーンショットを添付し、API やスキーマ変更時はマイグレーション手順と後方互換性リスクを記載します。
- 重大リグレッション懸念がある場合は Draft PR で早期相談し、追加の回帰テスト範囲を共有してください。

## Security & Configuration Tips
- `.env.example` を最新に保ち、Supabase Anon Key のみ共有。機微情報はコミット禁止です。
- Cloudflare Workers Secret は `wrangler secret put` で登録し、CORS は `https://*.word-omikuji.app` に限定します。
- 依存更新時は `pnpm audit --prod` と `pnpm dlx npm-check-updates` を使い、Breaking change は個別 PR で周知します。

---
> Source: [kami8ma8810/word-omikuji](https://github.com/kami8ma8810/word-omikuji) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
