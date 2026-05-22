## dnsweeper-ver2

> - Monorepo (Node.js 20, TypeScript, ESM).

# Repository Guidelines

## Project Structure & Module Organization
- Monorepo (Node.js 20, TypeScript, ESM).
- App: `packages/dnsweeper/`
  - Source: `src/` (CLI in `src/cli`, core in `src/core`)
  - Dist: `dist/cli/`
  - Tests: `tests/` (`unit/`, `integration/`, `e2e/`)
- Docs: `docs/` (SPEC, RISK_ENGINE, OPERATIONS, BENCHMARKS)
- Workflows: `.github/workflows/`
- Examples/presets and rules: `examples/`

## Build, Test, and Development Commands
- Build: `pnpm -C packages/dnsweeper run build` (tsup → `dist/cli`)
- Run CLI (dev): `npm start -- <args>` or `node packages/dnsweeper/dist/cli/index.js <args>`
- Unit tests: `pnpm -C packages/dnsweeper run test:unit`
- Network tests: `pnpm -C packages/dnsweeper run test:net`
- All tests: `pnpm -C packages/dnsweeper run test`
- Lint/Format: `pnpm -C packages/dnsweeper run lint` / `pnpm -C packages/dnsweeper run format`

Examples:
- Analyze: `npm start -- analyze packages/dnsweeper/sample.csv --http-check --doh --summary`
- Import+Export: `npm start -- import <csv> --pretty --output out.json && npm start -- export out.json --format csv --output out.csv`

## Coding Style & Naming Conventions
- TypeScript, strict ESM. 2-space indentation, no trailing spaces.
- Files: kebab-case (e.g., `risk-engine.ts`), classes/types: PascalCase, variables/functions: camelCase.
- Lint: ESLint (v9) + `@typescript-eslint`, configあり。Prettierで整形。

## Testing Guidelines
- Framework: Vitest。Coverage(V8) 目標は `vitest.config.ts` に準拠（lines/functions/statements≈80%、branches≈70%）。
- 置き場所: `tests/unit|integration|e2e`。命名は `*.test.ts`。
- 方針: Unitはネットワーク禁止。ネット依存は integration/e2e で。

## Commit & Pull Request Guidelines
- Commits: 簡潔な現在形（例: `feat: ...`, `fix: ...`, `chore: ...`）。関連変更を1コミットに。
- PRs: 目的/変更点/動作確認を記述。関連Issueをリンク。CLI出力変更はログやスクリーンショットを添付。
- CI: build/lint/unit はグリーン必須。ネット依存テストは必要に応じて実行。

## Security & Configuration Tips
- Node 20（`.nvmrc`）。ローカルは PNPM 推奨。
- 設定: `dnsweeper.config.json` で analyze/risk 既定値を管理。秘密情報のハードコード禁止。
- 監査ログ: マスキング/サイズ上限あり。機密や巨大データの出力は避ける。
- 安全策: `sweep apply` は GA まで封印。破壊的操作は計画ファイル＋レビュー必須。

## Architecture Overview (Quick)
- CLI（commander）: `analyze/import/export/list/ruleset/annotate/sweep/jobs` を登録。
- Core: DoH resolver, HTTP probe, Risk engine（R-001+）, ruleset adjuster, parsers, output。

---
> Source: [kazu-apps/dnsweeper.ver2](https://github.com/kazu-apps/dnsweeper.ver2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
