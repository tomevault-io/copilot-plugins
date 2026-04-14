## ksau-project

> **Last Updated:** 2026-02-28

# KSAU Project — AI Agent Universal Guide

**Last Updated:** 2026-02-28
**Current Phase:** AIRDP Cycle 24 (H61–H63 pending)

> 詳細なAIRDPロール手順はプロンプト（`airdp_prompts/`）で与えられる。

---

## 1. プロジェクト概要

**KSAU (Knot/String/Anyon Unified Framework)** は、標準模型粒子を3次元双曲結び目トポロジーにマッピングするTQFT。AIRDPサイクルで仮説を検証している。

### 確立済みの主要結果

| 成果 | 精度 | 根拠 |
| --- | --- | --- |
| κ = π/24（24-cell共鳴） | 誤差 0% | H6, H16, H36, H39, H44 |
| フェルミオン質量階層 | R²=0.9998 | H1, H11, H35, H41 |
| 重力定数 G の導出 | 誤差 0.000026% | H20, H46, H53 |
| 12粒子トポロジー割当規則 | p=0.0, 12/12 | H49, H55 |
| CKM 混合行列 | R²=0.9974 | v6.0 final |

### 現在の未解決課題（Cycle 23 REJECT）

- **H58**: 新規予測がランダム割当と統計的に区別不能 (p=0.067)
- **H59**: トーション補正 α=√2κ のLOO不安定 (LOO-R²=0.11 vs train R²=0.28)
- **H60**: det ≡ 0 (mod 24) が安定性と負の相関 (OR=0.745) — 理論的前提の矛盾

---

## 2. SSoT（Single Source of Truth）— 最重要ルール

**物理定数・理論値は必ず以下から読み込む。ハードコード禁止。**

```text
ssot/constants.json          ← 全理論定数・統計閾値
ssot/parameters.json         ← 粒子データ（質量・崩壊幅等）
ssot/data/raw/topology_assignments.json  ← トポロジー割当
ssot/hypotheses/H*.json      ← 仮説定義と判定結果
ssot/project_status.json     ← プロジェクト状態ダッシュボード
ssot/changelog.json          ← SSoT変更履歴
```

```python
# 正しい読み込み方
from ksau_ssot import SSOT
ssot = SSOT()  # パスは自動解決

# 禁止
kappa = 0.1309       # ハードコード禁止
up_mass = 2.16       # ハードコード禁止
```

**パス規則:** コード内のSSoTパスはプロンプトに明示された絶対パスを使用すること。プロンプトに記載がない場合はプロジェクトルートの `ssot/` を探すこと。`cycles/cycle_NN/` 内に `ssot/` を作成してはならない。

---

## 3. 統計的厳密性

すべての検証で以下を遵守する：

- **Bonferroni補正**: α = 0.05 / (サイクル内仮説数)。現在の標準閾値は p < 0.016667（3仮説/サイクル）
- **LOO-CV 必須**: 新しいモデルにはLeave-One-Out交差検証を実施。訓練誤差と検証誤差の両方を報告
- **FPR上限**: 通常 < 50%。精密予測は < 1%
- **モンテカルロ**: n=10000, seed=42（SSoTから読み込む）
- **自由パラメータ**: 観測量との比率を常に明示する

---

## 4. 科学的整合性

- **否定的結果を保護する**: REJECT は ACCEPT と同等に価値ある記録
- **事後的調整禁止**: 結果を見てからパラメータを調整するカーブフィッティングは即座に MODIFY
- **合成データ禁止**: `np.random.seed` による Ground Truth 生成は Reviewer が即座に却下
- **循環論法の検出**: 検証に使った基準で候補を選ぶことは禁止（例: DM候補60個のみで det基準を検証）
- **主張の範囲を守る**: LOO-CV で確認されていない精度を主張しない

---

## 5. プロジェクト構造

```text
KSAU_Project/
├── ssot/                        # Single Source of Truth（変更はJudge ACCEPT後のみ）
│   ├── constants.json           # 理論定数・統計閾値
│   ├── parameters.json          # 粒子データ
│   ├── changelog.json           # 変更履歴
│   ├── project_status.json      # プロジェクト状態ダッシュボード
│   ├── hypotheses/H*.json       # 仮説定義と判定結果
│   └── data/                    # KnotInfo/LinkInfo生データ
├── cycles/                      # AIRDPサイクル記録
│   └── cycle_NN/
│       ├── seed.md              # サイクルの出発点（人間が承認）
│       ├── roadmap.md           # Phase 2 Orchestratorが生成
│       ├── iterations/iter_NN/  # Researcher/Reviewerの作業
│       ├── verdict.md           # Phase 4 Judgeが生成
│       └── cycle_report.md      # Phase 5 Orchestratorが生成
├── airdp_prompts/               # AIロール別プロンプト
├── v6.0/                        # 公式リリース（Zenodo向け）
│   └── data/physical_constants.json  # 旧SSoT（v6.0コードのみ参照）
├── NEGATIVE_RESULTS_INDEX.md    # 否定的結果の保護リスト
└── idea_queue.md                # 次サイクル候補アイデア
```

---

## 6. AIRDPシステム概要

本プロジェクトは **AIRDP（AI Research Development Protocol）** で運用される。

| フェーズ | 役割 | 主な出力 |
| --- | --- | --- |
| Phase 2 | Orchestrator（計画） | `roadmap.md`, `ssot/hypotheses/H*.json` |
| Phase 3 | Researcher（実装）↔ Reviewer（審査） | `iter_*/results.json`, `iter_*/review.md` |
| Phase 4 | Judge（判定） | `verdict.md` |
| Phase 5 | Orchestrator（報告） | `cycle_report.md`, 次サイクル `seed.md`, SSoT更新 |

詳細は `airdp_prompts/` を参照。

---

## 7. 禁止事項（どのロールでも共通）

- `ssot/` 以外のパスへの定数のハードコード
- `cycles/cycle_NN/` 内への `ssot/` ディレクトリ作成
- 合成データによる Ground Truth 生成
- 検証済み判定の遡及的変更
- Judge の verdict を Orchestrator が独自に解釈・修正
- Bonferroni補正なしの多重検定

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yui-synth-lab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yui-synth-lab)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
