## unified-mcp

> OpenMemory と Cipher を束ねるラッパー MCP サーバー（Node.js）。

# unified-mcp

OpenMemory と Cipher を束ねるラッパー MCP サーバー（Node.js）。

## Language & Runtime

- JavaScript (CommonJS) / Node.js >= 18
- エントリポイントは `unified-mcp.js` の単一ファイル構成
- 依存は `mcp-remote` のみ
- リンター/フォーマッター: Biome (`biome.json` で設定)
- JSON 設定ファイルを編集する際は構文を検証してから保存すること

## Architecture

- OpenMemory: mcp-remote 経由の Streamable HTTP 接続（`OPENMEMORY_URL`）
- Cipher: stdio で子プロセス起動（`CIPHER_CMD`, `CIPHER_CWD`）
- 両サーバーのツールを統合して単一の MCP として公開
- 親プロセス終了時に子プロセスをプロセスグループごと kill（v0.3.6）

## Environment Variables

必須: `OPENMEMORY_URL`, `CIPHER_CMD`, `CIPHER_CWD`, `MCP_SERVER_MODE`, `USE_ASK_CIPHER`
任意: `NPX_PATH`, `CIPHER_AGENT_CONFIG`

## Agent Memory

- このプロジェクトは unified-memory MCP サーバーを使用している。コードベースやユーザーの好みについて重要なことを学んだら `add_memories` で保存し、複雑なタスクの開始時には `search_memory` で過去のコンテキストを検索すること

## Workflow

- 編集後は `npm run lint` で Biome チェックを通すこと
- 編集後は `node unified-mcp.js` で起動確認すること
- 既存ファイルの編集を優先し、不要なファイルを増やさない
- バージョンは `unified-mcp.js` 冒頭のコメントと `package.json` の両方を更新する

---
> Source: [shooebill/unified-mcp](https://github.com/shooebill/unified-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
