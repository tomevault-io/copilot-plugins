## claudecode-foxtamp-plugins

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

Claude Codeプラグインマーケットプレイス。Claude Codeにスキル（Confluenceアクセス等）を追加するプラグインを管理・配布する。

## アーキテクチャ

```
Marketplace (.claude-plugin/marketplace.json)
  └── Plugin (plugins/<plugin-name>/.claude-plugin/plugin.json)
        └── Skill (plugins/<plugin-name>/skills/<skill-name>/SKILL.md)
              └── Scripts (plugins/<plugin-name>/skills/<skill-name>/scripts/*.sh)
```

- **marketplace.json**: プラグイン一覧を宣言するマーケットプレイスルート設定
- **plugin.json**: 各プラグインのメタデータ（名前、説明、バージョン）
- **SKILL.md**: スキルの定義・トリガー条件・使用パターンをYAMLフロントマター付きMarkdownで記述
- **scripts/**: スキルの実装（Bashスクリプト）

## プラグイン追加手順

新規プラグインは既存の`confluence-plugin`と同じ構成に従う：

1. `plugins/<plugin-name>/.claude-plugin/plugin.json` を作成（name, description, version）
2. `plugins/<plugin-name>/skills/<skill-name>/SKILL.md` にスキル定義を記述
3. 必要に応じて `scripts/` ディレクトリにBashスクリプトを配置
4. `.claude-plugin/marketplace.json` の `plugins` 配列にエントリを追加

## 環境変数

| 変数名 | 必須 | 説明 |
|--------|------|------|
| `CONFLUENCE_PAT` | Yes | Confluence Personal Access Token（Bearer認証） |
| `CONFLUENCE_BASE_URL` | Yes | ConfluenceベースURL（例: `https://confluence.example.com`） |

## スクリプト実行

```bash
# ページIDで取得
plugins/confluence-plugin/skills/confluence-reader/scripts/fetch_confluence.sh <pageId> [expand_fields]

# URLから取得（ページID自動抽出）
plugins/confluence-plugin/skills/confluence-reader/scripts/fetch_by_url.sh <confluence_url> [expand_fields]
```

## 依存ツール

- `curl` — HTTPクライアント（自己署名証明書対応: `-k`フラグ使用）
- `jq` — JSONパーサー（推奨）
- `grep` — Perl正規表現サポート（`-oP`フラグ、URL解析に使用）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sinoa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sinoa)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
