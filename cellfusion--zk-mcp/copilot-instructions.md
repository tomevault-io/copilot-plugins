## zk-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`zk-mcp` is an MCP (Model Context Protocol) server written in Rust that exposes [zk-org/zk](https://github.com/zk-org/zk) Zettelkasten knowledge bases to Claude Desktop / Claude Code via stdio transport (JSON-RPC).

## Build & Run

```bash
cargo build              # Debug build
cargo build --release    # Release build
cargo run                # Run (stdio transport, blocks on stdin)
cargo test               # Run all tests
cargo test <test_name>   # Run a single test
cargo clippy             # Lint
cargo fmt                # Format
```

CLI args: `--notebook <path>` overrides notebook directory.
Env: `ZK_NOTEBOOK_DIR` sets notebook directory.

## Architecture

```
Claude Desktop/Code  ←— stdio (JSON-RPC) —→  zk-mcp (Rust)
                                                  │
                                        ┌─────────┴─────────┐
                                        │                    │
                                   zk CLI              直接 FS 読み書き
                                (検索/一覧/新規作成)     (ノート本文)
                                        │
                                     SQLite (zk DB)
```

- **zk CLI ラッパー (`zk_runner.rs`)**: 検索・タグ一覧は `zk list --format jsonl` で非同期実行。`--notebook-dir` を全コマンドに付与
- **直接ファイルアクセス**: ノート本文の読み書きは `tokio::fs` で実行（CLI オーバーヘッド回避）
- **zk 出力パース**: `zk list --format jsonl` で1行1JSON。`absPath` からノートブック相対パスを計算。`title` は空のことが多いので `lead` フィールドの frontmatter から抽出

### MCP Tools

Implemented (Phase 1-3):

| Tool | 目的 | 実装方式 |
|------|------|---------|
| `search_notes` | 全文検索 (FTS) | `zk list --match` |
| `read_note` | ノート本文読み取り + バックリンク | `tokio::fs` + frontmatter パース + `zk list --link-to` |
| `list_tags` | タグ一覧 + 使用回数 | `zk tag list --format json` |
| `list_notes` | フィルタ付きノート一覧 | `zk list --format jsonl` |
| `get_recent_notes` | 最近のノート取得 | `zk list --created-after --sort created-` |
| `get_backlinks` | リンクグラフ (incoming/outgoing) | `zk list --link-to` / `--linked-by` |
| `create_note` | ノート新規作成 + TagPipeline | `zk new --print-path` + `tokio::fs` |
| `append_note` | ノート追記 | `tokio::fs::OpenOptions (append)` |
| `notebook_health` | ヘルスチェック (stubs/orphans/tagless) | `zk list` + Rust フィルタ |
| `tag_stats` | タグ統計 + 共起分析 | `zk tag list` + 全ノート分析 |
| `find_related_context` | キーワード関連検索 | `zk list --match` + スコアリング |

### MCP Prompts

Implemented (Phase 4):

| Prompt | 目的 | 引数 |
|--------|------|------|
| `daily_review` | 今日のノート要約・タスク抽出 | なし |
| `knowledge_lookup` | トピック横断検索 | `topic: String` |
| `note_from_conversation` | 会話内容をノートに構造化 | なし |
| `weekly_retrospective` | 週間振り返り・ナレッジ成長分析 | なし |
| `project_context` | プロジェクト関連ナレッジロード | `project_keywords: Vec<String>` |

### MCP Resources

| URI | 種別 | 内容 |
|-----|------|------|
| `zk://tags` | 静的 | タグ一覧 + 使用回数 |
| `zk://recent` | 静的 | 最近のノート (1週間) |
| `zk://recent/today` | 静的 | 今日のジャーナルエントリ |
| `zk://health` | 静的 | ノートブックヘルスサマリー |
| `zk://stats` | 静的 | タグ統計サマリー |
| `zk://tags/{tag_name}` | テンプレート | 特定タグのノート一覧 |
| `zk://note/{path}` | テンプレート | 特定ノートの内容 |

### CLI サブコマンド

| コマンド | 目的 |
|---------|------|
| `zk-mcp init-claude` | CLAUDE.md + .mcp.json テンプレート生成 |

### Tag Pipeline (`src/tags/`)

ノート作成・更新時のタグ処理は3段パイプライン:
1. **Normalizer** — ルールベース正規化 (小文字化、ハイフン統一、階層整理)
2. **Alias Map** — 表記揺れ解決 (`typescript` → `ts`)、設定ファイルでカスタマイズ可能
3. **Validator** — 既存タグとの照合、Jaro-Winkler ファジーマッチで類似候補提示

### Module Structure

```
src/
├── main.rs           # エントリポイント、サーバー起動
├── server.rs         # MCP ServerHandler + #[tool_router] で全ツール定義
├── zk_runner.rs      # zk CLI ラッパー (search_notes, list_tags, list_notes)
├── config.rs         # 設定ファイル (~/.config/zk-mcp/config.toml)
├── frontmatter.rs    # YAML frontmatter パーサー
├── error.rs          # ZkMcpError エラー型
└── tags/             # タグ正規化・バリデーション
    ├── mod.rs        # TagPipeline (統合)
    ├── normalizer.rs # 正規化ルール 5種
    ├── alias.rs      # エイリアスマップ
    └── validator.rs  # Jaro-Winkler ファジーマッチ
```

## rmcp SDK Usage Notes

- `rmcp` v1.3: `#[tool_router]` on impl block, `#[tool]` on methods, `#[tool_handler]` on `ServerHandler` impl
- Tool methods return `Result<String, String>` or `String` (implements `IntoCallToolResult`)
- `CallToolResult` does NOT implement `IntoCallToolResult` directly; use `Result<String, String>` instead
- Parameters via `Parameters<T>` wrapper, NOT `#[tool(aggr)]`
- Transport: `rmcp::transport::io::stdio()` returns `(Stdin, Stdout)`
- All logging to stderr via `tracing` (stdout is MCP channel)

## Tech Stack

- **MCP SDK**: `rmcp` v1.3 (公式 Rust SDK)
- **Async runtime**: `tokio`
- **CLI 実行**: `tokio::process::Command`
- **Fuzzy match**: `strsim` (Jaro-Winkler)
- **Serialization**: `serde` / `serde_json` / `serde_yaml`
- **Error handling**: `anyhow` / `thiserror`

## Configuration

設定ファイル: `~/.config/zk-mcp/config.toml`

主要設定: `notebook_dir` (zk ノートブックのパス), タグエイリアス, 正規化ルール, ファジーマッチ閾値

## Design Documents

- `docs/SPEC.md` — 全体設計書 v2 (MCP Tools/Resources/Prompts 定義、実装フェーズ)
- `docs/TAG.md` — タグバリデーション・正規化モジュール設計

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cellfusion)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cellfusion)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
