## sawapc-shio-motioncontroller

> teleop-hand — 指間距離計測値によるロボットハンド遠隔操作システム

# CLAUDE.md

teleop-hand — 指間距離計測値によるロボットハンド遠隔操作システム

## プロジェクト概要

finger-tracker が計測した指間距離（mm）を指令値として受信し、ロボットハンドの把持動作をリアルタイムに遠隔操作するシステム。将来的にバイラテラルテレオペレーション（力フィードバック）への発展を目指す。

既存の controller_try（C++ リアルタイムモーション制御フレームワーク）をベースに、通信層・制御ロジック・ハードウェアI/Oを整理・改修して構築する。

姉妹プロジェクト: [finger-tracker](../finger-tracker/) — RealSense D435i + YOLOv8-nano による指間距離リアルタイム計測

## プロジェクト構造

```
teleop-hand/
├── CLAUDE.md                    # プロジェクト概要・開発ルール
├── CMakeLists.txt               # ビルド設定
├── config/                      # 設定ファイル（JSON）
│   ├── system.json              # システム全体設定
│   └── joints/                  # 関節パラメータ（モータごと）
├── inc/                         # ヘッダファイル
├── src/                         # ソースファイル
│   ├── main.cc                  # エントリポイント
│   ├── comm/                    # finger-tracker との通信
│   ├── controller/              # 制御ロジック（位置制御・力制御・バイラテラル）
│   ├── hardware/                # ハードウェアI/O（Contec DA/カウンタ）
│   ├── config/                  # 設定読み込み・管理
│   └── gui/                     # GUI表示（OpenGL）
├── scripts/                     # ユーティリティスクリプト
├── lib/                         # サードパーティライブラリ（Eigen等）
├── tests/                       # テスト
├── data/                        # 実行時データ（git管理外）
├── logs/                        # ログ（git管理外）
└── docs/                        # ドキュメント体系
```

## ビルド・テスト

```bash
# ビルド
mkdir -p build && cd build
cmake ..
make

# 実行（root権限が必要 — PCIデバイスアクセス）
sudo ./control

# テスト（予定）
make test
```

## 技術スタック

| 層 | 技術 | 用途 |
|---|---|---|
| 言語 | C++ | メイン言語 |
| ビルド | CMake | ビルドシステム |
| 線形代数 | Eigen | 行列演算 |
| GUI | ImGui + ImPlot + GLFW + OpenGL | ライブ状態表示・グラフ・モード切替 |
| ハードウェアI/O | Contec PCI DA/カウンタボード | モータ電圧出力・エンコーダ読み取り |
| JSON | Boost.PropertyTree | 設定ファイル読み込み |
| 通信 | UDP ソケット (POSIX socket) | finger-tracker からの指令値受信（[ADR 002](docs/design/decisions/002-communication-protocol.md)） |

技術選定の詳細: [docs/design/decisions/001-technology-stack.md](docs/design/decisions/001-technology-stack.md)

## アーキテクチャ

### モジュール構成

| モジュール | 責務 | 依存先 |
|-----------|------|--------|
| **comm** | finger-tracker との通信。指間距離（mm）を受信して制御ループに渡す | config |
| **controller** | 制御ロジック。位置制御・力制御・バイラテラル制御等のモード切替と実行 | config, comm, hardware |
| **hardware** | ハードウェアI/O。Contec DA ボードへの電圧出力、カウンタボードからのエンコーダ読み取り | config |
| **config** | 設定管理。システム設定・関節パラメータ（JSON）の読み込み | — |
| **gui** | GUI表示。制御モード切替UI、関節状態のリアルタイム表示 | controller, hardware |
| **scripts** | ユーティリティ。Contec ボード有効化スクリプト等 | — |

### 処理フロー

```
finger-tracker（別プロセス）
    │ 指間距離 (mm)
    ▼
comm（通信層）── UDP受信（localhost:50000, ADR 002）
    │
    ▼
controller（制御ロジック）── DOBベース加速度制御（ADR 004）
    │ 制御モード: idle / DA_check / remote / Record / Bilateral
    │
    ▼
hardware（ハードウェアI/O）
    ├── DA ボード → モータ電圧出力
    └── カウンタ ← エンコーダパルス読み取り
```

### 制御ループ

controller_try から継承するリアルタイム制御構造（[ADR 004](docs/design/decisions/004-control-architecture.md)）:

- タイマースレッドによる周期実行（サンプリング周波数: 設定ファイルで指定、デフォルト 10kHz）
- 関節ごとの状態管理: command / response / reference / compensation / output
- 信号処理: 擬似微分 → DOB（外乱オブザーバ）→ 制御則 → トルクリミット → 電圧変換
- 制御モード: idle / DA_check / remote（余弦定理逆運動学+PD制御, [ADR 005](docs/design/decisions/005-control-philosophy.md)） / Record（remote+データ記録） / Bilateral（将来、EMS力フィードバック）

## 用語

| 用語 | 定義 |
|------|------|
| 指間距離 | finger-tracker が計測する親指-人差指間の3D距離（mm） |
| DOB | 外乱オブザーバ (Disturbance Observer)。外乱を推定・補償する制御手法 |
| RFOB | 反力推定オブザーバ (Reaction Force Observer)。環境からの反力を推定する |
| EMS | 電気筋肉刺激 (Electrical Muscle Stimulation)。操作者への力フィードバックに使用（Phase 3） |
| バイラテラル制御 | マスター（人間の手）とスレーブ（ロボットハンド）間で力と位置を双方向に伝達する制御方式 |
| DA ボード | デジタル-アナログ変換ボード。制御信号（電圧）をモータに出力する |
| カウンタボード | エンコーダパルスを計数するボード。モータの回転角度を取得する |
| joint | 1つのモータ（関節）の状態・パラメータを管理する単位 |
| controller_try | ベースとなる既存のリアルタイムモーション制御フレームワーク（`/home/enosawa/lab/controller_try/`） |

## 実装ルール

- controller_try のコードを直接コピーせず、モジュール構成に合わせて整理・改修して取り込む
- ハードウェア依存コードは `hardware/` モジュールに集約する。制御ロジック (`controller/`) はハードウェア非依存とする
- 設定値（関節パラメータ、サンプリング周波数等）はハードコードせず JSON 設定ファイルで管理する
- 命名規則は controller_try に準拠: snake_case（ファイル名、変数、関数、クラス名、メソッド）、メンバ変数は末尾アンダースコア
- namespace `mc` を使用する
- `data/`, `logs/` は git 管理外

### コミットメッセージ

形式: `type(scope): 簡潔な説明`

```
feat(comm): finger-tracker との通信モジュールを追加
feat(controller): 位置制御モードの実装
fix(hardware): DA ボード初期化のタイミング修正
```

## タスク管理ルール

### 基本設定

タスクリストID `teleop-hand-tasks` で統一（`.claude/settings.json`で設定済み）。

### サブエージェント起動

必ず `model: "sonnet"` を指定する。

```
Task {
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: "..."
}
```

## ドキュメントインデックス

| ドキュメント | 内容 |
|-------------|------|
| [docs/archive/initial_plan.md](docs/archive/initial_plan.md) | 初期仕様書 |
| [docs/design/decisions/](docs/design/decisions/) | ADR一覧 |
| [docs/design/GUIDE.md](docs/design/GUIDE.md) | 設計書作成ガイド |
| [docs/plans/GUIDE.md](docs/plans/GUIDE.md) | 実装計画ガイド |
| [docs/review/GUIDE.md](docs/review/GUIDE.md) | レビューガイド |
| [docs/usecases/GUIDE.md](docs/usecases/GUIDE.md) | ユースケースガイド |
| [docs/status/implementation.md](docs/status/implementation.md) | 実装ステータス |
| [docs/status/roadmap.md](docs/status/roadmap.md) | ロードマップ |

---
> Source: [sasakishion0620/sawaPC_shio_MotionController](https://github.com/sasakishion0620/sawaPC_shio_MotionController) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
