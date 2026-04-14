## esa-mcp-server

> esa.ioのAPIを使用して、基本的な操作を行えるMCPサーバーを作る

# プロジェクト

esa.ioのAPIを使用して、基本的な操作を行えるMCPサーバーを作る
.envで完了変数を管理し、ESA_TOKEN, ESA_TEAM_NAMEでそれぞれAPIトークンとチーム名を管理する。

esa.ioのAPIについては https://docs.esa.io/posts/102 を参照。

# 使用技術

- deno
- TypeScript

# 開発計画

1.  **基盤構築フェーズ**
    *   Denoプロジェクト初期化 (`deno init`, `biome.json` 設定)
    *   環境変数設定 (`.env`, Denoでの読み込み)
    *   APIクライアント基礎 (認証、ベースURL)
    *   Memory Bank 初期設定 (`projectbrief.md`, `techContext.md`)

2.  **コア機能実装フェーズ**
    *   **ユーザー情報取得 (`/user`, テスト)**
    *   記事一覧取得 (`/posts`, 検索、テスト)
    *   記事詳細取得 (`/posts/:post_number`, テスト)
    *   エラー処理
    *   Memory Bank 更新 (`systemPatterns.md`, `activeContext.md`, `progress.md`)

3.  **追加機能と改善フェーズ**
    *   記事作成 (`POST /posts`, テスト)
    *   記事更新 (`PATCH /posts/:post_number`, テスト)
    *   記事削除 (`DELETE /posts/:post_number`, テスト)
    *   ドキュメント作成 (`.ray_local/cursor_docs/`)
    *   Memory Bank 更新 (`productContext.md`)

# 進め方のポイント

*   フェーズごとに `git commit`
*   TDD (テスト駆動開発)
*   **コードを修正したら型情報のチェック(`deno check`)と、テスト (`deno test`) とフォーマット/リントチェック (`deno fmt`, `deno lint` or Biome commands) を実行するのだ**
*   必要に応じて機能追加 (コメント等)
*   サーバーのドキュメント作成

# Logging in FastMCP Tools

- **DO NOT** use `console.log()` or `console.error()` for logging within MCP
  tool `execute` functions.
- **ALWAYS** use the `log` object destructured from the second argument of the
  `execute` function (e.g., `async execute(args, { log })`) for logging
  messages.
  - Use `log.info()`, `log.warn()`, `log.error()`, `log.debug()` as appropriate.

# FastMCP Server Development Notes

## Logging

- **DO NOT** use `console.log()` or `console.error()` anywhere in the MCP server process, including imported modules (`src/esa_client.ts`, etc.). Using `console` in the server process interferes with the stdio transport used by Cursor and will likely cause `Client closed` errors.
- **ALWAYS** use the `log` object destructured from the second argument of the tools `execute` function for logging within tools (e.g., `async execute(args, { log })`). Use `log.info()`, `log.warn()`, `log.error()`, etc.
- For logging outside of `execute` (e.g., during server initialization), standard logging methods provided by `FastMCP` seem unavailable. If necessary, consider temporary file logging (`Deno.writeTextFile`) with `--allow-write`, but remove it for final use if possible.

## MCP Server Reloads

- After modifying the MCP server code (`main.ts` or its dependencies) by adding/removing tools or changing server settings, Cursor needs to **reload** the MCP server information.
- If a tool appears as "not found" immediately after adding it, try reloading the MCP server connection in Cursor.
- Sometimes multiple reloads might be necessary for changes to be fully recognized.

## Environment Variables & Imports

- Be cautious with code that runs immediately upon module import (like top-level environment variable checks in `src/esa_client.ts`).
- If such code uses `console` or performs actions like `Deno.exit()`, it can prevent the MCP server from starting correctly and cause `Client closed` errors even before any tools are called.
- Ensure necessary environment variables (`ESA_TOKEN`, `ESA_TEAM_NAME`) are correctly set (e.g., via `.env` and `dotenv`) before starting the server.

## Other Notes (Add as needed)

# Prioritize Lint/Format Errors

Address linter and formatter errors immediately when they arise. Do not proceed with other tasks until these errors are resolved.

# Periodically Check Formatting and Linting

Remember to periodically run `deno task check` (or equivalent command like `deno fmt && deno lint`) to ensure code quality and consistency.

# Added Tasks (by Gemini)

## `check:all`

- **Command:** `deno task check:all`
- **Purpose:** Runs formatting (`deno fmt`), lint fixing (`deno lint --fix`), and type checking (`deno check`) sequentially.
- **Usage:** Use this task regularly to ensure overall code quality before committing or pushing changes.

# Linting Rules (Added by Gemini)

## `no-unused-vars`

- **Action:** Delete unused imports or variables completely.
- **DO NOT:** Use underscore prefixes (e.g., `_variable`) or comment out the unused code.

## General Unused Code

- **Action:** Remove any code that is not currently used or planned for immediate use.
- **Reasoning:** Keep the codebase clean and focused.

## 定数定義

定数を定義する際は `as const` を使用して、より厳密な型推論を有効にするのだ。

```typescript
// Good
export const MY_OPTIONS = {
  optionA: 'valueA',
  optionB: 123,
} as const;

// Bad
export const MY_OPTIONS = {
  optionA: 'valueA',
  optionB: 123,
};
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masseater) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
