## multiproject-research

> This file serves as a persistent reference for the data locations and models used in the current analysis, specifically for the IRL vs. RF comparison and prediction disagreement analysis.

# GEMINI Reference Data

This file serves as a persistent reference for the data locations and models used in the current analysis, specifically for the IRL vs. RF comparison and prediction disagreement analysis.

## 1. Random Forest (RF) Model
- **Model Name:** `rf_nova_cross_eval_unified_rs42`
- **Results Directory:** `outputs/rf_nova_cross_eval_unified_rs42/`
- **Key Metrics Files:**
    - `matrix_AUC_ROC.csv`
    - `matrix_F1.csv`
    - `matrix_PRECISION.csv`
    - `matrix_RECALL.csv`

## 2. Inverse Reinforcement Learning (IRL) Model
- **Model Name:** `review_continuation_cross_eval_nova`
- **Results Directory:** `results/review_continuation_cross_eval_nova/`
- **Key Metrics Files:**
    - `matrix_AUC_ROC.csv`
    - `matrix_F1.csv` (Note: sometimes named `matrix_f1_score.csv` in older runs, but `matrix_F1.csv` exists here)
    - `matrix_PRECISION.csv`
    - `matrix_RECALL.csv`

## 3. Prediction Disagreement Analysis (IRL vs. RF)
- **Data Directory:** `@Bachelor2025_Hashimoto/discussion/prediction_analysis/`
- **Scope:** Restricted to 10 valid patterns where `train_period` <= `test_period`.
- **Key Data Files:**
    - `both_correct.csv`: Cases where both models predicted correctly.
    - `both_wrong.csv`: Cases where both models predicted incorrectly.
    - `irl_only_correct.csv`: Cases where only IRL predicted correctly.
    - `rf_only_correct.csv`: Cases where only RF predicted correctly.
    - `irl_only_unique_developers.csv`: Analysis of unique developers correctly predicted only by IRL.

## 4. Source Data (Raw)
- **Review Requests:** `data/review_requests_openstack_multi_5y_detail.csv`
    - Used for analyzing reviewer distribution, acceptance rates, and historical features.

## 5. Generated Visualizations
- **Directory:** `@Bachelor2025_Hashimoto/Hashimoto_fig/discussion/`
- **Key Figures:**
    - `venn_irl_rf.pdf`: Venn diagram showing the overlap of correct predictions.
    - `venn_patterns/`: Subdirectory containing Venn diagrams for each of the 10 train/test period patterns.
    - `heatmap_2x2.pdf`: confusion matrix heatmap (RF Correct/Wrong vs IRL Correct/Wrong).
    - `boxplot_features_combined.pdf`: Multi-panel boxplots comparing response time, review load, and activity gap across prediction categories.
    - `scatter_history_acceptance.pdf`: Scatter plot of history count vs. acceptance rate (Input features used for prediction).
    - `scatter_history_acceptance_split.pdf`: 真のラベル（継続/離脱）別に分割した散布図。離脱者の「左下領域」におけるIRLの優位性を可視化。
    - `acceptance_rate_bins_stacked.pdf`: Stacked bar chart of acceptance rate bins over time.
    - `acceptance_rate_boxplot.pdf`: Boxplot of acceptance rates by period.
    - `median_trends.pdf`: Trends of median acceptance rate and response delay.

## 6. Scripts
- **Visualization Script:** `@Bachelor2025_Hashimoto/src/visualize_prediction_disagreement.py`
    - Generates the Venn diagram, heatmap, and scatter plots based on the analysis data.
    - **実行方法:** 依存関係解決のため `uv run` を使用すること。
        ```bash
        uv run @Bachelor2025_Hashimoto/src/visualize_prediction_disagreement.py
        ```

## 7. Analysis Summary (2026-01-29)
- **IRL vs RF の不一致分析結果:**
    - 離脱した開発者（Label=0）において、IRLのみが正解したケースは「レビュー依頼数が中程度（数十〜百件程度）だが承諾率が低い」領域（散布図の左下）に集中している。
    - RFは履歴数の多さに引きずられて「継続」と誤予測しやすいが、IRLは承諾率の低さや活動の質の変化を報酬関数を通じて学習することで、より正確に離脱を予測できている。
    - 継続した開発者（Label=1）については両手法とも高い精度を維持しており、特に安定した貢献者については RF も有効である。
    - 軸ラベルや図の構成を整理し、真のラベル別の分割散布図（`scatter_history_acceptance_split.pdf`）を用いることで、IRLの優位性を視覚的に明確化した。

## 8. General Instructions
- **Response Language:** 最終的な分析結果や回答は日本語で行うこと。
- **Execution Tool:** Pythonスクリプトの実行には `uv run` を優先的に使用すること。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kzk03)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kzk03)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
