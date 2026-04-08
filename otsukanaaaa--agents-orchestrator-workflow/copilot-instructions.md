## agents-orchestrator-workflow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 重要: 言語設定 / IMPORTANT: Language Settings

**Claude Code は必ず日本語で応答してください。**

- すべてのメッセージは日本語で返答すること
- コードのコメントも日本語で記述すること
- エラーメッセージの説明も日本語で行うこと
- 技術用語は必要に応じて英語併記可

**Claude Code MUST respond in Japanese.**

- All messages must be in Japanese
- Code comments should be in Japanese
- Error explanations should be in Japanese
- Technical terms can include English when necessary

## プロジェクト概要 / Project Overview

自然言語タスク記述から自動的にエージェント組織を生成し実行するAIエージェント自動生成・実行システム。タスクを分析し、最適なエージェント構造を設計し、マルチエージェント協調を通じて複雑なタスクを実行します。

This is an AI Agent Generation and Execution System that automatically creates agent organizations from natural language task descriptions. The system analyzes tasks, designs optimal agent structures, and orchestrates multi-agent collaboration to execute complex tasks.

## 開発コマンド / Development Commands

### 環境セットアップ / Environment Setup

```bash
# uvを使用した依存関係のインストール（推奨）
uv sync

# OpenAI APIキーの設定（LangGraphコンポーネントに必要）
export OPENAI_API_KEY="your-api-key"
# または.envファイルを作成: OPENAI_API_KEY=your-api-key
```

### システムの実行 / Running the System

```bash
# メインプログラムの実行
uv run python main.py
```

### 開発ツール / Development Tools

```bash
# テストランナーは未設定
# pyproject.tomlにリンター/フォーマッターは未設定
# 推奨: ruff, black, mypy（コード品質向上のため）
```

### Jupyter Notebookの利用

```bash
# 開発用環境のインストール
uv sync --group dev

# Jupyter Notebookの起動
uv run jupyter notebook
```

## システムアーキテクチャ / System Architecture

階層的なマルチエージェントシステムによる自動タスク実行パイプライン：

### コアパイプラインフロー / Core Pipeline Flow

1. **タスク分析 (Task Analysis)**
   - LLMAnalyzerが自然言語タスクを分析
   - タスクの複雑度評価、フェーズ分割、必要スキルの特定
   - 実行計画（TaskPlan）の自動生成

2. **実行計画の階層構造**
   - **TaskPlan**: 最上位のタスク実行計画
     - 複数のPhase（フェーズ）で構成
   - **Phase**: タスクの実行フェーズ
     - 複数のSubTask（サブタスク）で構成
   - **SubTask**: 最小実行単位のタスク

3. **エージェント自動生成と実行**
   - **TaskExecutor**: タスク全体を管理・実行
   - **PhaseExecutor**: 各フェーズを実行
   - **SubTaskExecutor**: 個別のサブタスクを実行
   - 各レベルでLangChainベースのReActエージェントを動的生成

4. **階層的な結果集約**
   - SubTaskResult → PhaseResult → TaskResult
   - 各レベルでの結果を統合して最終結果を生成

## プロジェクト構造 / Project Structure

```
agent-generator/
├── main.py                  # エントリーポイント
├── src/
│   ├── models.py           # データモデル定義（Pydantic）
│   ├── agent_organization.py # メインシステムクラス
│   ├── llm_analyzer.py     # LLMベースのタスク分析
│   ├── task_executor.py    # タスク実行エンジン
│   └── agent_collaboration.py # （未使用）
├── jupyter/                # Notebook実験環境
│   └── main.ipynb
└── pyproject.toml         # プロジェクト設定

```

## 主要な依存関係 / Key Dependencies

- **langchain** & **langchain-openai**: LLMインテグレーション
- **langgraph**: エージェントグラフオーケストレーション（ReActエージェント生成）
- **pydantic**: データバリデーションとモデル定義
- **python-dotenv**: 環境変数管理
- **python-ulid**: ユニークID生成
- **tavily-python**: Web検索ツール（将来の拡張用）

## 環境変数 / Environment Variables

- `OPENAI_API_KEY`: 必須 - OpenAI APIアクセス用

## 実装の特徴 / Implementation Features

### 自動タスク分析
- GPT-4oを使用した高度なタスク理解
- タスクの複雑度評価と最適な分割
- 必要スキルの自動識別

### 階層的エージェント構造
- 3層のエージェント階層（Task → Phase → SubTask）
- 各レベルで独立したエージェントが動作
- 上位エージェントが下位の結果を統合

### 柔軟な拡張性
- ツールの追加が容易（LangChainツール統合）
- 新しいエージェントタイプの追加が可能
- カスタムプロンプトテンプレートのサポート

## 使用例 / Usage Example

```python
from src.agent_organization import AgentOrganization, SystemConfig, TaskRequest

# システム設定
config = SystemConfig(llm_model="gpt-4o-2024-08-06")
agent_org = AgentOrganization(config)

# タスク実行
task = "ユーザーの購買履歴を分析して、商品推薦レポートを作成してください"
request = TaskRequest(task_id="task-001", description=task)
result = await agent_org.execute_task(request)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/otsukanaaaa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/otsukanaaaa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
