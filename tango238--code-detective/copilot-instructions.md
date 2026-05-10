## code-detective

> Linear イシューを自動調査する Python デーモン。OpenAI Symphony のアーキテクチャパターン（Poll-Dispatch-Execute）を Claude API で再実装。

# Investigation Service

## What this project is

Linear イシューを自動調査する Python デーモン。OpenAI Symphony のアーキテクチャパターン（Poll-Dispatch-Execute）を Claude API で再実装。

## Key documents

- `SPEC.md` — 全体技術仕様。実装の根拠は全てここにある。
- `WORKFLOW.md` — ランタイム設定（YAML front matter）+ 調査プロンプト（Markdown body）。
- `HANDOFF.md` — 設計セッションでの決定事項と経緯。「なぜこうなったか」はここを参照。

## Architecture summary

```
main.py (asyncio daemon)
  → orchestrator.tick() every 30s
    → tracker/linear.py: Linear GraphQL で "Investigating" イシューを取得
    → classifier.py: イシューを分類（Phase 1 は全て source_code）
    → agents/source_code.py: Claude API tool_use ループで調査実行
      → tools/bash_exec.py: ホワイトリスト方式でシェルコマンド実行
      → tools/mysql_query.py: SELECT のみ許可
      → tools/read_file.py: ワークスペース内ファイル読み取り
    → 結果を Linear コメントとして投稿
    → イシューを "Human Review" に遷移
```

## Critical design decisions

1. **No HTTP server** — FastAPI は不要。pure daemon + structlog。
2. **Claude API, not Claude Code CLI** — subprocess ではなく `anthropic` SDK の tool_use ループ。
3. **Security in tool executor** — bash はホワイトリスト、MySQL は SELECT のみ、パストラバーサル防止。
4. **In-memory state** — DB 不要。再起動時は Linear を再読み込みして復旧。
5. **Phase 1 = source_code only** — Classifier は実質スタブ。enabled_categories で拡張。

## Code style

- Python 3.12+, asyncio ベース
- Type hints 必須（dataclass, Protocol 活用）
- structlog でイベントログ（print 禁止）
- テストは pytest + pytest-asyncio

## Common commands

```bash
# 開発実行
python -m src.main

# テスト
pytest tests/ -v

# 型チェック
mypy src/

# リント
ruff check src/
```

---
> Source: [tango238/code-detective](https://github.com/tango238/code-detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
