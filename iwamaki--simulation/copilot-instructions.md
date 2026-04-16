## simulation

> This file provides guidance to Gemini when working with code in this repository.

# GEMINI.md

This file provides guidance to Gemini when working with code in this repository.

## Project Overview

原子・分子スケールのシミュレーション統合環境。MD（分子動力学）とDFT（第一原理計算）をPythonで管理し、構造構築・計算実行・結果解析を行う。

## Environment

- **Python**: 3.12 (`.venv/` に仮想環境)
- **LAMMPS**: `/home/iwash/lammps/build/lmp` (ソースビルド版)
- **Quantum ESPRESSO**: `/home/iwash/qe/build/bin/` (v7.4, ソースビルド版)
- **DAMASK**: 3.0.2 (PPA版: `ppa:damask-multiphysics/ppa`)
- **Platform**: WSL2 (Ubuntu 24.04) on Windows
- **RunPod**: クラウドGPUによるリモート実行環境（後述）

### Setup & Run

```bash
source .venv/bin/activate

# 汎用ツール
python scripts/<script_name>.py

# テーマ固有のシミュレーション（プロジェクトルートから実行）
python simulations/<テーマ>/<driver>.py [options]
```

## Architecture

```
scripts/               # 汎用ユーティリティ（テーマに依存しない）
potentials/            # ポテンシャルファイル（MD用）
notebooks/             # Jupyter Notebook（解析・可視化用）
simulations/
  └── <テーマ>/         # テーマ単位で管理
      ├── <driver>.py   #   実行ドライバ（構造構築→入力生成→実行）
      ├── PROGRESS.md   #   テーマ固有の進捗・知見メモ
      └── <条件名>/      #   個別条件の結果フォルダ（自動生成）
```

### scripts/ と simulations/ の役割分担

- **`scripts/`**: どのテーマでも使える汎用ツール（プロット、デバッグ、テスト等）
- **`simulations/<テーマ>/`**: テーマ固有のドライバスクリプト＋出力データを同一フォルダに格納。テーマ固有の知見は `PROGRESS.md` に記録する。

### Key Libraries

- **ASE (`ase`)**: 結晶構造構築、各種計算コードへの入出力
- **NumPy/SciPy**: 数値計算・データ処理
- **Matplotlib**: グラフ作成

## RunPod（クラウドGPU実行）

大規模計算はRunPod上のGPUインスタンスでリモート実行できる。ドライバスクリプトに `--runpod --gpu` を付けるだけで、Pod作成→ファイル転送→GPU実行→結果ダウンロード→Pod停止が自動で行われる。

```bash
# ローカル実行（CPU）
python simulations/<テーマ>/<driver>.py

# RunPod GPU実行
python simulations/<テーマ>/<driver>.py --runpod --gpu
```

- **Dockerイメージ**: `ghcr.io/iwamaki/lammps:latest`（`docker/Dockerfile.lammps`）
  - LAMMPS + GPU package (CUDA) でビルド
  - 対応GPU: A100 (SM80), RTX 3070/3090/A4000 (SM86), RTX 4090/L40 (SM89), H100 (SM90)
  - イメージ再ビルドなしでGPU種別を切り替え可能
- **実行基盤**: `scripts/runpod_runner.py`（SSH経由でジョブ投入・結果回収）
- **環境変数**: `RUNPOD_API_KEY`（必須）、`RUNPOD_GPU_TYPE`、`RUNPOD_IMAGE`（`.env`で管理）
- **オプション**: `--keep-pod`（連続実行時にPodを維持してコスト削減）

## Conventions

- コメント・ドキュメントは日本語で記述
- ドライバスクリプトはプロジェクトルートから実行する前提（`os.getcwd()` でルートを参照）
- テーマ固有の設定・Tips・進捗は各 `simulations/<テーマ>/PROGRESS.md` に記録する

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iwamaki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
