## paper-curator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

arXiv または Hugging Face Trending から論文を取得し、Claude API を使ってユーザの興味に基づいてフィルタリングし、Obsidian 用の Markdown ファイルとして出力するツール。

## コマンド

```bash
# 依存パッケージのインストール
pip install -r requirements.txt

# 実行：arXiv モード（--output は必須）
python main.py --days 7 --output /path/to/obsidian/inbox -v

# 実行：Hugging Face Trending モード
python main.py --hf --output /path/to/obsidian/inbox -v

# 個別モジュールのテスト
python src\paper_curator\arxiv_client.py
python src\paper_curator\hf_client.py
python src\paper_curator\llm_filter.py
python src\paper_curator\output.py
```

## 環境変数

- `ANTHROPIC_API_KEY`: Claude API キー（必須）

## アーキテクチャ

```
main.py (CLI エントリーポイント)
    │
    ├─→ arxiv_client.fetch_papers()         # arXiv API から論文取得（arXiv モード）
    ├─→ hf_client.fetch_trending_papers()   # HF Trending から論文取得（--hf モード）
    │
    ├─→ output.extract_existing_paper_ids() # 既存ファイルから紹介済みID抽出
    │   └─→ 紹介済み論文を除外
    │
    ├─→ llm_filter.load_interests()         # YAML から興味プロファイル読込
    ├─→ llm_filter.filter_papers()          # Claude API でスコアリング
    │
    └─→ output.generate_markdown()          # Markdown ファイル生成
```

データフロー: arXiv/HF Trending → 論文リスト → 紹介済み除外 → Claude API でフィルタ → Markdown 出力

出力ファイル:
- arXiv モード: `papers-YYYY-MM-DD.md`
- HF Trending モード: `papers-YYYY-MM-DD-hf.md`

## コーディング規約

- README.md およびソースコメントは英語で記述
- `config/interests.yaml`（興味プロファイル）は日本語可
- 複数行文字列は `textwrap.dedent()` を使用してインデントを揃える

## 設定ファイル

`config/interests.yaml`:
- `categories`: 取得する arXiv カテゴリのリスト
- `interests.primary`: 主要な興味領域
- `interests.strong_interest`: 特に興味があるもの
- `interests.low_interest`: 興味が薄いもの

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tsuyoshimishina)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tsuyoshimishina)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
