## hocbf-grasp-sim

> hocbf_grasp_sim — controller_try ベースの HOCBF-QP 安全把持制御

# CLAUDE.md

hocbf_grasp_sim — controller_try ベースの HOCBF-QP 安全把持制御

## プロジェクト概要

controller_try（研究室共有リアルタイム制御フレームワーク）をベースに、HOCBF-QPによる力センサレス安全把持制御モードを追加するプロジェクト。`#define SIMULATOR` でシミュレーション、`#define PCI_MODE` で実機として動作する。

滑落防止（力の下限）と変形防止（位置の上限）の二重安全制約をHOCBF-QPで保証。RFOBによるオンライン剛性推定に基づく適応的安全集合と、Graspability Index（把持可能性指標）を実装する。

## プロジェクト構造

```
hocbf_grasp_sim/
├── CLAUDE.md
├── CMakeLists.txt              # controller_try ベース
├── config/                     # JSONパラメータファイル
│   ├── system.json             # sample_frequency等
│   ├── blue_scara/             # 関節設定（controller_try由来）
│   │   └── 4018_finger.json    # 把持モータ（M=0.000054, g_dis=600）
│   └── objects/                # 物体別パラメータ（新規）
│       ├── pet_bottle_full.json
│       └── ...
├── inc/                        # ヘッダファイル
│   ├── robot_system.h          # controller_try フレームワーク
│   ├── joint.hpp               # 関節データ構造
│   ├── signal_processing.h     # DOB, LPF, 疑似微分
│   ├── simulator.h             # シミュレータ（拡張: 物体モデル組込）
│   ├── controller.h            # 制御モード登録
│   ├── mode_definition.h       # HOCBF_grasp モード追加
│   ├── hocbf_qp.h              # 【新規】HOCBF-QP (u_lb, u_ub, clamp)
│   ├── rfob.h                  # 【新規】RFOB (K̂_obj推定)
│   ├── adaptive_theta_max.h    # 【新規】θ_max適応則
│   ├── contact_detector.h      # 【新規】接触検出（二重条件AND）
│   ├── object_model.h          # 【新規】弾塑性遷移物体モデル
│   └── ...                     # controller_try の既存ヘッダ
├── src/                        # ソースファイル
│   ├── main.cc                 # エントリポイント（SIMULATOR/PCI_MODE切替）
│   ├── controller.cc           # 全制御モード（HOCBF_grasp を追加）
│   └── ...                     # controller_try の既存ソース
├── lib/                        # Eigen, imgui等（controller_try由来）
├── scripts/
│   ├── plot.py                 # 【新規】Matplotlib でCSV→プロット
│   └── enable_contec_board.sh  # controller_try由来
├── data/                       # CSV出力先
└── docs/                       # ドキュメント
```

## ビルド・テスト

```bash
# ビルド（シミュレーションモード）
mkdir -p build && cd build
cmake ..    # CMakeLists.txt で -DSIMULATOR が有効
make

# 実行
sudo ./control

# GUIでモード選択:
#   HOCBF_grasp → 把持制御開始
#   HOCBF_grasp_data → データ記録

# 可視化
python3 scripts/plot.py data/result.csv
```

## 技術スタック

| 層 | 技術 |
|---|---|
| ベース | controller_try フレームワーク（robot_system, joint, thread_controller, GUI） |
| 言語 | C++17 |
| ビルド | CMake |
| JSON | Boost.PropertyTree |
| GUI | OpenGL + imgui |
| 信号処理 | signal_processing.h（DOB, LPF, 疑似微分） |
| 可視化 | Python3 + Matplotlib |

技術選定の詳細: [docs/design/decisions/001-technology-stack.md](docs/design/decisions/001-technology-stack.md)

## アーキテクチャ

### 新規追加ヘッダと責務

| ヘッダ | 責務 |
|--------|------|
| **hocbf_qp.h** | u_lb, u_ub 計算、clamp、Graspability Index |
| **rfob.h** | K̂_obj = f̂_dis / θ_rel + LPF |
| **adaptive_theta_max.h** | K̂_obj変化率判定 → θ_max 更新（α_relax/α_shrink） |
| **contact_detector.h** | DOB外乱の適応的閾値 AND K̂_obj閾値 |
| **object_model.h** | 弾塑性遷移: K_obj(θ) = K_0 × exp(-β(θ-θ_yield)) |

### 制御モード（controller.cc）

```cpp
controller[mc::HOCBF_grasp] = [](robot_system &robot) {
    // Phase 0: PD位置制御 + 接触検出
    // Phase 2: CBF-QP + RFOB + 適応則
    // Phase 3: 定常保持
};
```

### 処理フロー（Phase 2, 1ステップ）

```
エンコーダ → 疑似微分(θ̇) → DOB(f̂_dis) → RFOB(K̂_obj)
                                              ↓
                                    adaptive_theta_max → θ_max(t)
                                              ↓
u_nom(PD制御) → hocbf_qp(u_lb, u_ub) → u* = clamp(u_nom, u_lb, u_ub)
                                              ↓
                                         モータ出力
```

## 用語

| 用語 | 定義 |
|------|------|
| HOCBF | High-Order Control Barrier Function。相対次数2以上の安全制約を扱うCBF |
| QP | Quadratic Programming。1-DOFではclamp関数に帰着 |
| DOB | Disturbance Observer。外乱トルクを推定 |
| RFOB | Reaction Force Observer。環境反力から接触剛性K̂_objを推定 |
| K̂_obj | RFOBで推定した物体の接触剛性 [Nm/rad] |
| θ_max(t) | 適応的に更新される最大許容変形角 [rad] |
| τ_min | 滑落防止に必要な最小把持トルク [Nm] |
| Graspability Index (G) | G = u_ub - u_lb。G > 0 なら安全な入力が存在する |

## 実装ルール

- controller_try の規約に従う: snake_case、メンバ変数末尾アンダースコア
- HOCBF固有のロジックは inc/ のヘッダに分離。controller.cc のラムダは薄いラッパー
- 既存マクロ（`x_res(n)`, `f_out(n)`, `M(n)` 等）を使用
- 新規パラメータは `robot.set_to_dict()` / `robot.get_from_dict()` または config JSON で管理

### コミットメッセージ

形式: `type(scope): 簡潔な説明`

```
feat(hocbf): HOCBF-QP安全フィルタを実装
feat(rfob): RFOB剛性推定を実装
feat(adaptive): θ_max適応則を実装
fix(contact): 接触検出の閾値を調整
```

## タスク管理ルール

### 基本設定

タスクリストID `hocbf-grasp-sim-tasks` で統一（`.claude/settings.json`で設定済み）。

### サブエージェント起動

必ず `model: "sonnet"` を指定する。

## ドキュメントインデックス

| ドキュメント | 内容 |
|-------------|------|
| [docs/design/decisions/001-technology-stack.md](docs/design/decisions/001-technology-stack.md) | 技術スタック（controller_tryベース） |
| [docs/design/decisions/002-system-architecture.md](docs/design/decisions/002-system-architecture.md) | 3リポジトリ構成（hocbf_grasp_sim/tau_min_estimation/controller_try） |
| [docs/design/decisions/003-simulation-design.md](docs/design/decisions/003-simulation-design.md) | シミュレーション設計（モード追加方式・フェーズ・CSV仕様） |
| [docs/design/decisions/004-object-model.md](docs/design/decisions/004-object-model.md) | 物体モデル（弾塑性遷移・5物体パラメータ） |
| [docs/design/decisions/005-safety-parameters.md](docs/design/decisions/005-safety-parameters.md) | 安全制約パラメータ（γ, 適応則, 全初期値一覧） |
| [docs/design/decisions/006-tau-min-estimation.md](docs/design/decisions/006-tau-min-estimation.md) | τ_min推定パイプライン（別リポ・JSON連携） |
| [docs/design/decisions/007-observer-design.md](docs/design/decisions/007-observer-design.md) | オブザーバ設計（DOB/RFOB・高周波注入・フィルタゲイン） |
| [docs/design/decisions/008-contact-detection.md](docs/design/decisions/008-contact-detection.md) | 接触検出（二重条件AND・フェーズ遷移） |
| [docs/plans/001-simulation-implementation.md](docs/plans/001-simulation-implementation.md) | 実装計画（8タスク・5マイルストーン） |
| [docs/archive/initial_plan.md](docs/archive/initial_plan.md) | 初期仕様書 |
| [docs/design/GUIDE.md](docs/design/GUIDE.md) | 設計書作成ガイド |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sawa-E)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sawa-E)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
