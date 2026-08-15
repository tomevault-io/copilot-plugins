## initiation

> HENKAKU Initiation。docs/development-plan.md が全体計画、docs/superpowers/plans/ が実装計画。

# AGENTS.md

HENKAKU Initiation。docs/development-plan.md が全体計画、docs/superpowers/plans/ が実装計画。

- Node.js 22.0.0 以上 / npm(バージョン管理ツールは各自の好みで)
- テスト: `npm test`(Vitest)
- 環境変数は .env.local(コミット禁止)。変数名一覧は .env.example
- 決定事項は `docs/decisions/` に1決定1ファイルで記録。命名規則は `docs/decisions.md` を参照
- CI: PRとmainへのpushで `.github/workflows/ci.yml` が検証コマンドを実行する
- 依存更新: Dependabotの更新PRは、CIが緑でも自動マージしない。ウォレット・ブラウザ依存のフローはCIの対象外なので、`wagmi` / `viem` / `next` を含む更新は `/setup` の接続と署名を手で確認してからマージする(`docs/decisions/2026-08-10-dependency-updates.md`)
- Dependabotのnpm更新で`npm ci`がlockfile不整合になった場合は、CIを緩めず`docs/decisions/2026-08-11-dependabot-lockfile-recovery.md`の一時復旧手順に従う

## Work selection

Treat `docs/development-plan.md` as the source of truth for product direction. Open Issues are a backlog, not a priority list.

Unless explicitly requested otherwise, choose work that directly advances the current phase's completion condition. Prefer core user and operator flows over infrastructure, refactoring, documentation, design polish, and future-phase work.

Before implementation, explain how the change advances the completion condition. If a product decision is missing, prepare the smallest decision aid or prototype and ask for direction. If no core work can proceed, report the blocker instead of switching to an easier peripheral Issue.

## ページ構成

- `/setup`: ウォレット接続、SIWEサインイン、Polygon切替、HENKAKU追加
- `/initiation`: 質問・クエストの進捗保存と完走判定
- `/checkin`: 1日1回のチェックインと履歴
- `/apply`: Allowlist追加・HENKAKU配布の申請と状態表示
- `/admin`: `ADMIN_ADDRESSES` に登録された管理者向けの申請審査・状態更新

## ローカル検証コマンド

```bash
# 開発サーバー(既存のプロセスを確認してから起動する)
npm run dev -- --port 3000

# 単体・統合テスト
npm test

# 型チェック / lint / 本番build
# クローン直後はbuildが生成する型ができるまでtscが失敗するため、build を先に実行する
npm run build
npx tsc --noEmit
npm run lint

# ローカルSupabase
npx supabase start
npx supabase status
npx supabase db reset  # ローカルDBを初期化する場合のみ
```

統合テストは `.env.local` のローカルSupabase設定を自動で読み込む(`vitest.config.ts`)。CI等では従来どおり環境変数で渡してもよい。キーの値をログやコマンド出力に表示しない。本番Supabaseへのmigration適用とVercelデプロイは、運用入力と公開前チェックが確定してから行う。

## テスト配置

- `tests/unit/`: Server Action、ドメイン、認証ガードなどの単体テスト
- `tests/integration/`: ローカルSupabaseへ接続するRepositoryテスト
- `tests/support/`: 統合テスト用のDB初期化・補助コード
- Vitestの対象は `tests/**/*.test.ts`

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---
> Source: [henkaku-center/initiation](https://github.com/henkaku-center/initiation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
