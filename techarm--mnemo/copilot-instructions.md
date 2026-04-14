## mnemo

> このプロジェクトは Claude Code 用のナレッジ管理システムです。

# Mnemo プロジェクト

このプロジェクトは Claude Code 用のナレッジ管理システムです。
ユーザーが手書きした内容はここに書きます。

<!-- MNEMO:START - この部分は Mnemo が自動生成します。手動で編集しないでください -->
<!-- 最終更新: 2026-03-08 14:23 -->

## プロジェクト情報
- **名前:** mnemo
- **説明:** Knowledge memory system for Claude Code
- **言語:** typescript
- **技術スタック:** typescript, lancedb

## 👤 User Profile

- **Identity:** name: Techarm, role: Full-stack developer
- **Technical:** languages: TypeScript, Python, os: macOS
- **Tools:** packageManager: pnpm, linter: oxlint, formatter: prettier
- **Communication:** language: 日本語, style: concise
- **Coding Style:** naming: camelCase for vars, PascalCase for types

## 📖 Project Docs

### System Design

- **システムアーキテクチャ** (`.claude/docs/architecture.md`): 3テーブル構成（knowledge/projects/tasks）、LanceDB+Ollama、MCP+CLIデュアルインターフェース、ファイルベースドキュメント、ユーザープロフィール、procedureタイプ

### Features

- **ハイブリッド検索** (`.claude/docs/hybrid-search.md`): Vector検索+FTS→Reciprocal Rank Fusion→多次元スコアリング（semantic 0.6, bm25 0.25, recency 0.1, confidence 0.05）
- **信頼度減衰システム** (`.claude/docs/confidence-decay.md`): 時定数τ=180日の指数減衰、フロア値0.1、recall時+0.1ブースト、起動時バッチ処理、procedureタイプは減衰対象外
- **CLAUDE.md 自動生成** (`.claude/docs/claude-md-generation.md`): マーカーベース部分更新、confidence>=0.5フィルタ、知識・タスク・ドキュメントをセクション分類して出力、procedure/referenceセクション追加

### API / Interfaces

- **MCPツール・CLIインターフェース** (`.claude/docs/mcp-tools.md`): 11個のMCPツール（learn/recall/project/task/doc/generate/backup/stats/export/delete/profile）+ CLI + 7スキル（learn/research/setup/session-review/doc/code-reuse-finder） + フック

_5件のドキュメント。詳細は `.claude/docs/` を Read tool で参照。_

## ⚠️ Pitfalls（既知の落とし穴）
- **LanceDB query()のデフォルトlimitは10件**: LanceDBのtable.query().toArray()はデフォルトでlimit(10)が適用される。全件取得するにはquery().limit(10000).toArray()のように明示的にlimitを指定する必要がある。countRows()は正しい件数を返すがquery()はデフォルト10件しか返さないため、11件以上のデータがある場合にデータが欠落する。
- **LanceDB where句でcamelCaseカラム名にはバッククォートが必要**: LanceDBのwhere句でcamelCaseのカラム名（例: projectId, createdAt, updatedAt）を使う場合、そのままだと小文字に変換されて 'No field named projectid' エラーになる。バッククォートで囲む必要がある。例: where(`projectId` = 'value') は正しい。where("projectId" = 'value
- **LanceDBのFTSインデックスはデータ追加後に再構築が必要** _(グローバル)_: LanceDBではデータを追加した後、FTSインデックスを replace: true オプションで再作成しないと新しいデータが全文検索にヒットしない。addKnowledgeEntry の後に rebuildFtsIndexes を呼ぶ必要がある。

## 📐 Patterns（確立されたパターン）
- **LanceDBスキーマ進化は読み取り時正規化で対応** _(グローバル)_: LanceDBは新カラムを自動受け入れするが、古い行の新カラムはundefinedになる。normalizeKnowledgeEntry()パターンで読み取り時にデフォルト値（空文字列/0）を埋めることで、ALTER TABLE不要で後方互換を維持できる。スキーマバージョンを上げるたびにこのパターンを適用する。手順: (1) 型定義に新フィールド追加, (2) schema.tsのplacehol

## 💡 Lessons（教訓）
- **Obsidian CLIはbatch exportに不向き** _(グローバル)_: Obsidian v1.12.4で公式CLIが追加されたが、実行中のObsidianアプリへのリモコン（remote control）として設計されている。主な制約: (1) バッチ操作コマンドなし — 1ノートずつ個別にcreateする必要がある、(2) importコマンドなし、(3) フォルダ作成コマンドなし、(4) frontmatterの設定はcreateとは別コマンド（properti

## 📚 References（参照知識）
- **OpenClaw ワークスペース構成と設計思想**

<!-- MNEMO:END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/techarm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/techarm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
