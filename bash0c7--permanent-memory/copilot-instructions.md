## permanent-memory

> Ruby MCP server. esa.io `bist` チーム・tag `永続` をバックエンドとする永続記憶。

# permanent-memory

Ruby MCP server. esa.io `bist` チーム・tag `永続` をバックエンドとする永続記憶。

## Tools

- `permanent_memory_search` — 検索（`tag:永続 -in:Archived`）
- `permanent_memory_create` — 新規作成（タグ `永続` 自動付与）
- `permanent_memory_update` — 更新（revision 競合検知あり）

実装: `scripts/mcp_server.rb`

## Skills（Claude Code のみ）

- `/permanent-memory-archive` — ポストをアーカイブして記憶から除外
- `/permanent-memory-organize` — 永続記憶ポストの整理・重複排除

## Dev

```sh
bundle install   # 依存: mcp gem, test-unit
```

認証・設定の詳細は README.md 参照。

---
> Source: [bash0C7/permanent-memory](https://github.com/bash0C7/permanent-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
