## rss-feed

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ホワイトリスト式RSS統合システム。`config.yaml`で指定したソース（YouTubeチャンネル、汎用RSS）からフィードを取得し、日付順にソートしたRSS 2.0形式で出力。GitHub Actionsで定期実行し、GitHub Pagesで配信。

## Commands

```bash
# 依存インストール
uv sync

# RSS生成実行
uv run python -m app.main

# テスト実行
uv run pytest tests/ -v

# リンター
uv run ruff check .

# フォーマット
uv run ruff format .

# pre-commitフック手動実行
uv run pre-commit run --all-files
```

## Architecture

```
config.yaml → app/main.py → docs/feed.xml → GitHub Pages
                  ↓
           SourceFetcher (抽象基底クラス)
                  ├── YouTubeFetcher
                  └── GenericRSSFetcher
```

### Key Components

- `app/config.py`: YAML設定読み込み、`AppConfig`/`FeedConfig`/`SourceConfig`データクラス定義
- `app/models.py`: `NormalizedItem`データクラス（各ソースからの正規化済みアイテム）
- `app/sources/base.py`: `SourceFetcher`抽象基底クラス
- `app/sources/youtube.py`: YouTubeチャンネルRSS取得（公式RSS、APIキー不要）
- `app/sources/generic_rss.py`: 汎用RSS/Atomフィード取得
- `app/feed_builder.py`: `NormalizedItem`リストからRSS 2.0 XML生成
- `app/main.py`: CLIエントリポイント、ソース取得→統合→出力の流れを制御

### Data Flow

1. `load_config()` で `config.yaml` を読み込み
2. 有効な各ソースに対して適切な `SourceFetcher` を生成
3. `fetcher.fetch()` で `NormalizedItem` リストを取得
4. `FeedBuilder.build()` で全アイテムをマージ・ソート・XML出力

### Error Handling

- 個別ソースの失敗は他ソースに影響しない
- 全ソース失敗時は既存の `feed.xml` を保持（上書きしない）

## Source Types

| type | required field | description |
|------|---------------|-------------|
| `youtube_channel` | `channel_id` | YouTubeチャンネル公式RSS |
| `generic_rss` | `rss_url` | 任意のRSS/Atomフィード |

## Pre-commit Hooks

- gitleaks: シークレット漏洩防止

---
> Source: [Lettuce222/rss-feed](https://github.com/Lettuce222/rss-feed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
