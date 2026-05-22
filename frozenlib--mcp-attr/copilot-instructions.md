## mcp-attr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

mcp-attrは、Model Context Protocol (MCP) サーバーを宣言的に構築するためのRustライブラリです。属性マクロを使用してMCPサーバーを簡単に作成できるように設計されています。

### Architecture

このプロジェクトは3つのワークスペースメンバーで構成されています：

- **mcp-attr**: メインライブラリ。MCPサーバーとクライアントの実装
- **mcp-attr-macros**: プロシージャルマクロの実装 (`#[mcp_server]`, `#[tool]`, `#[resource]`, `#[prompt]`)
- **codegen**: スキーマ生成とコード生成用のユーティリティ

### Key Components

- `#[mcp_server]` 属性によるMCPサーバーの宣言的記述
- `#[tool]`, `#[resource]`, `#[prompt]` 属性による機能の実装
- `#[complete_fn]` 属性による補完機能の実装（追加引数対応）
- MCPクライアント（テスト用）
- 型システムを活用したスキーマ生成

### Completion Function Enhancement (`#[complete_fn]`)

`#[complete_fn]` 属性に追加引数機能を実装しました：

#### 基本的な使用方法
```rust
#[complete_fn]
async fn complete_with_args(&self, value: &str, category: &str, count: Option<u32>) -> Result<Vec<String>> {
    // 実装
}
```

#### 対応している引数型
- `&str`: 必須文字列引数
- `Option<&str>`: オプショナル文字列引数
- `T: FromStr`: 必須の型変換可能な引数
- `Option<T: FromStr>`: オプショナルの型変換可能な引数

#### 引数の動作
- 引数は `CompleteRequestParams::context::arguments` (BTreeMap) から取得
- オプショナル引数がない場合は `None` を返す
- 必須引数がない場合は空の補完結果を返す
- 型変換に失敗した場合はエラーを返す
- 型互換性チェックはコンパイル時に行われる

#### テスト
- 統合テスト: `tests/completion_*.rs` のマトリックス構造テスト
- コンパイル失敗テスト: `tests/compile_fail/completion_*.rs`

## Development Commands

### Build and Test
```bash
# 全パッケージのビルド
cargo build

# 特定パッケージのビルド
cargo build -p mcp-attr
cargo build -p mcp-attr-macros
cargo build -p mcp-attr-codegen

# テスト実行
cargo test

# 特定パッケージのテスト
cargo test -p mcp-attr

# ドキュメントテスト
cargo test --doc

# コンパイル失敗テスト（trybuild）
cargo test --test compile_fail -- --ignored
```

### Code Quality
```bash
# 型チェック
cargo check

# テストのコンパイルチェック（実行なし）
cargo test --no-run

# Clippy（リンター）
cargo clippy

# 自動修正
cargo clippy --fix --allow-dirty

# ドキュメント生成
cargo doc

# フォーマット
cargo fmt
```

### Examples
```bash
# サンプル実行
cargo run --example char_count
cargo run --example tool_info
```

## Testing Strategy

### Test Organization
- `tests/` ディレクトリ: 統合テスト
- `tests/mcp_server_*.rs`: `#[mcp_server]` 属性のテスト（型ごとに1ファイル）
- `tests/compile_fail/`: コンパイル失敗テスト（trybuild使用）
- モジュール内 `tests` モジュール: 非公開項目のテスト

### Test Guidelines
- 新機能実装時は必ずテストを作成
- 複数テスト追加時は1つずつ追加して確認
- テストデータは英語を使用（非ASCII文字テスト時を除く）

### Debugging `#[mcp_server]` Macro
マクロのデバッグ時：
1. `#[mcp_server]` を `#[mcp_server(dump)]` に変更
2. テスト実行でマクロ展開後コードを確認
3. 展開後コードを直接編集してデバッグ
4. 修正内容をマクロ実装に反映

## Completion Tests Structure

### Test File Organization

Completion functionality tests are organized in a matrix structure based on:
- **Usage context**: prompt vs resource
- **Definition location**: global (global functions) vs impl (methods in #[mcp_server] impl)

#### Success Case Tests
```
tests/
├── completion_prompt_global.rs    # Prompt + Global completion functions
├── completion_prompt_impl.rs      # Prompt + Impl completion methods  
├── completion_resource_global.rs  # Resource + Global completion functions
├── completion_resource_impl.rs    # Resource + Impl completion methods
└── completion_edge_cases.rs      # Special cases not covered by the matrix
```

#### Compile Failure Tests
Located in `tests/compile_fail/` with naming pattern: `completion_[category]_[error].rs`

### Adding New Completion Tests

#### For Common Functionality
When adding a new test for functionality that applies to all completion contexts:
1. **MUST add the test to all 4 matrix files** (completion_prompt_global, completion_prompt_impl, completion_resource_global, completion_resource_impl)
2. Use consistent test naming across all files
3. Only exclude from specific files if functionality is genuinely incompatible
4. Document any exclusions with clear comments

#### For Special Cases
Use `tests/completion_edge_cases.rs` for cross-context integration tests, manual overrides, and cases that don't fit the matrix structure.

## Code Style

### Rust Conventions
- Rustの慣例とベストプラクティスに従う
- 関数名・型名は一貫性と対称性を重視
- 理解困難なコードのみにコメント付与
- バグ以外でErrが返されない場合はResultを使わずパニック

### Error Handling
- `mcp_attr::Result` と `mcp_attr::Error` を使用
- `bail!` (プライベート) と `bail_public!` (パブリック) マクロを活用
- 依存関係のエラーはプライベート情報として扱う

### Documentation
- 公開項目には適切なdocコメントを付与
- 最初の行は簡潔な1行説明
- 関連する型・関数は `[]` でリンク
- `cargo test --doc` と `cargo doc` で検証

## Dependencies

### Main Dependencies
- `serde`: JSON シリアライゼーション
- `tokio`: 非同期ランタイム
- `schemars`: JSON Schema生成
- `jsoncall`: JSON-RPC実装
- `uri-template-ex`: URI Template処理

### Development Dependencies
- `trybuild`: コンパイル失敗テスト
- `pretty_assertions`: テストアサーション

## Documentation Generation

### tests_readme.rs (Auto-generated File)

**IMPORTANT**: `mcp-attr/src/tests_readme.rs` is an auto-generated file and should NOT be edited directly.

- **Source**: Generated from `README.ja.md` (Japanese README)
- **Generation Command**: `rustdoc-include --root /path/to/project`
- **When to Regenerate**: 
  - After modifying README.md or README.ja.md
  - When doctest examples need updating
  - When completion function examples change

### Workflow for Updating Documentation Examples

1. Edit the source README file (`README.md` for English, `README.ja.md` for Japanese)
2. Run `rustdoc-include --root .` to regenerate tests_readme.rs
3. Run `cargo test --doc` to verify all doctests pass
4. Any direct edits to tests_readme.rs will be overwritten on next generation

### rustdoc-include Usage

```bash
# Regenerate tests_readme.rs from README files
rustdoc-include --root .
```

This command processes files with `#![include_doc("filename", start/end)]` markers and generates documentation tests.

## Important Notes

- 依存関係のバージョンダウンは禁止
- カレントディレクトリ変更は避け、コマンド引数で対応
- エラー修正3回失敗時はスキップして他の箇所を修正
- 依存関係の追加・変更は禁止とし、必要な場合はユーザーによる手動編集を促すこと
- **tests_readme.rsは自動生成ファイルのため直接編集しない** - README.mdを編集してrustdoc-includeで再生成する

---
> Source: [frozenlib/mcp-attr](https://github.com/frozenlib/mcp-attr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
