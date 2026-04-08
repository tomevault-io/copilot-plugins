## bt2

> このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

脳腫瘍病理画像解析プロジェクトです。WSI（Whole Slide Images）を用いた機械学習・深層学習による病理診断支援システムの開発を目標としています。1,721症例のデータセットを活用し、髄膜腫分類、良悪性判別、グリオーマ分子予測等の研究テーマに取り組みます。

## 環境構築

このプロジェクトではPythonパッケージ管理に `uv` を使用します：

```bash
# 依存パッケージのインストール
uv add <package-name>

# スクリプトの実行
uv run python <script-name>.py
```

## 主要データ構造

### コアデータセット
- `data/Patho2Diagnosis/`: 36,773件の病理診断記録を含むExcelファイル（2017-2024年）
- `data/WSI/`: 年度別に整理されたNDPI画像ファイル、1,821症例で13,257枚の画像
- `data/wsi_pathology_dataset.csv/xlsx`: WSI画像と診断テキストの両方を持つ1,721症例の統合データセット

### データスキーマ
統合データセットの必須カラム：
- `case_id` (病理番号): WSIと診断をマッチングする主キー
- `original_diagnosis`: 元の病理組織診断（平均106文字）
- `who5_primary`: WHO 5th edition準拠の主分類
- `vlm_class`: VLM学習用の最終分類ラベル
- `task1_class`, `task2_class`, `task3_class`: タスク別分類ラベル
- `idh_status`, `codeletion_1p19q`: 分子病理学的マーカー
- `wsi_path`, `wsi_file_count`: WSIメタデータ
- `split`: データ分割（train/val/test）

## アーキテクチャコンポーネント

### データ分析・統合スクリプト
- `analyze_pathology_data.py`: 診断テキストデータの基本統計
- `analyze_wsi_files.py`: WSIファイル構造と染色種別の分析
- `analyze_diagnosis_breakdown.py`: 診断内容の分類分析
- `extract_all_diagnoses.py`: 全診断の抽出と分析
- `design_classification_hierarchy.py`: 階層的分類システム設計
- `reclassify_with_who_criteria.py`: WHO 5th edition基準による再分類
- `create_balanced_vlm_classes.py`: VLM学習用最終データセット作成

### 主要データパターン
- WSI症例IDの形式: `YY-NNNN`（通常）または `NXX-NNN`（Nプレフィックス）
- 複数行症例（同じ`病理番号`）は診断を連結して統合
- HE染色は97.5%の症例で利用可能、MIB-1は79%の症例で利用可能
- 全病理症例のうちWSI画像があるのは約5%のみ

## 開発ワークフロー

### VLM学習用データセット生成
最終的なVLM学習用データセットを生成：
```bash
uv run python create_balanced_vlm_classes.py
```

### タスク別データセット
```bash
# Task 1: メイン分類（9クラス、1,721症例）
data/task1_major_categories.csv

# Task 2: グリオーマ分子分類（5クラス、243症例）  
data/task2_glioma_molecular.csv

# Task 3: WHO Grade分類（4クラス、764症例）
data/task3_who_grade.csv
```

### データ分析
```bash
uv run python analyze_diagnosis_breakdown.py  # 診断内容分析
uv run python extract_all_diagnoses.py        # 全診断抽出
uv run python reclassify_with_who_criteria.py # WHO再分類
```

## 重要な制約

### データの制限
- 現在のデータセット: 1,721症例（完全なVLM訓練には不十分）
- 推奨最小要件: プロダクションモデルには15,000-20,000症例
- 基本的な診断タスクに限定（良悪性分類で約80%の精度を期待）

### ファイル処理
- WSI画像は大きなNDPIファイル（gitで追跡しない）
- Excelファイルは日本語テキストを含むため適切なエンコーディングが必要
- 複数行統合ロジックは現在のデータセットで4つのエッジケースを処理

## モデル開発戦略

WHO 5th edition準拠分類と階層的学習アプローチ：

### 推奨実装順序
1. **Task 1 (Major Categories)**: 9クラス分類（1,721症例）- 最もバランスが良い
2. **Task 3 (WHO Grade)**: 4クラス分類（764症例）- 補助情報として有用
3. **Task 2 (Glioma Molecular)**: 5クラス分類（243症例）- 臨床価値最高だが最困難

### 技術的アプローチ
- 事前学習済み病理モデルによる転移学習
- HE染色画像を中心とした学習（97.5%カバレッジ）
- 分子病理学的マーカー（IDH, 1p19q）を考慮した分類
- WHO 5th edition準拠のグリオーマ分類（87.7%高信頼度）

詳細な開発推奨事項については `vlm_pathology_plan.md` と `wsi_pathology_matching_report.md` を参照してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/endaaman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/endaaman)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
