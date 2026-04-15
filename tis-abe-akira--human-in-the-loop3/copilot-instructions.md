## human-in-the-loop3

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

このプロジェクトは、LangGraphを使用したHuman-in-the-loopエージェントの実装です。住宅ローンの計算ツールを例にして、ツールの実行前に人間による承認を求める仕組みを実装しています。

## 起動方法

```bash
# アプリケーションを起動
uv run streamlit run app.py
```

## アーキテクチャ

### 主要コンポーネント

- **agent.py**: Human-in-the-loopエージェントの実装
  - `HumanInTheLoopAgent`: メインのエージェントクラス
  - `amortization_calculation`: 住宅ローンの計算ツール
  - StateGraphを使用した状態管理とワークフロー制御

- **app.py**: Streamlitを使ったWebUIの実装
  - メッセージ履歴の表示
  - ツール実行の承認インターフェース
  - エージェントとの対話機能

### 状態フロー

1. **call_llm**: LLMにリクエストを送信
2. **human_review_node**: ツール実行前の人間による承認待機
3. **run_tool**: 承認されたツールを実行

### ツールの実行フロー

- ツールの呼び出しが発生すると、自動的に`human_review_node`で停止
- UIに承認ボタンが表示され、ユーザーが承認またはリジェクト可能
- 承認後にツールが実行され、結果がLLMに送信される

## 重要なメソッド

- `handle_human_message()`: ユーザーメッセージの処理
- `handle_approve()`: ツール実行の承認
- `get_messages()`: 会話履歴の取得
- `is_next_human_review_node()`: 承認待ち状態の判定

## 依存関係

- langchain-openai: LLMとの統合
- langgraph: グラフベースのワークフロー
- streamlit: Web UI
- python-dotenv: 環境変数管理

## テスト方法

サンプル質問:
```
年利2.4%で30万円を36回払いするとどうなる？
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tis-abe-akira)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tis-abe-akira)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
