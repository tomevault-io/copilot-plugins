## board-company

> このプロジェクトに取りかかる前に、以下のドキュメントを順に読んでください。

# CLAUDE.md — AI アシスタント向けプロジェクトガイド

## 最初に読むドキュメント

このプロジェクトに取りかかる前に、以下のドキュメントを順に読んでください。

1. **[docs/11_プロジェクト構成.md](docs/11_プロジェクト構成.md)** — ディレクトリ構造、ファイル一覧、アーキテクチャ図、依存関係ルール
2. **[docs/10_技術設計.md](docs/10_技術設計.md)** — 疎結合設計の方針、core/ と views/ の分離原則
3. **[docs/01_ゲーム概要.md](docs/01_ゲーム概要.md)** — ゲームルール・勝利条件・基本仕様

変更対象の機能に応じて、関連する仕様書（docs/02〜09）も参照してください。

## プロジェクト概要

Pyxel 2.x で動作するレトロスタイルのボードゲーム（会社経営シミュレーション）。
Python 3.9+、256x256px / 16色 / 30FPS。外部アセット（.pyxres）は使わず全てプロシージャル描画。

## アーキテクチャ上の重要ルール

- **`src/core/` は Pyxel に依存しない。** `import pyxel` を core/ 配下に追加しないこと。
- **`src/views/` は差し替え可能。** BoardViewBase / PlayerViewBase の抽象クラスを実装する形で描画を行う。
- **データは JSON 駆動。** ゲームバランスの変更は `data/*.json` で行い、コードに直書きしない。
- **定数は `src/core/rules.py` に集約。** マジックナンバーを散在させない。

## 開発コマンド

```bash
# 依存インストール
pip install -r requirements.txt

# ゲーム起動（ローカル）
python main.py

# 自動テスト（pyxel-mcp が必要）
pip install pyxel-mcp
python tests/test_screens.py

# .pyxapp ビルド（ソース変更後は必ず再ビルド）
pyxel package . main.py

# Web 版 HTML の更新（.pyxapp 再ビルド後に実行）
pyxel app2html board-company.pyxapp
```

## MCP サーバー

`.mcp.json` に pyxel-mcp の設定がある。pyxel-mcp を使うと:

- ヘッドレスでゲームを実行しスクリーンショットを撮影できる
- キー入力をシミュレートしてゲーム進行を自動テストできる
- スプライト、タイルマップ、画面レイアウトを検査できる

動作確認時は `python tests/test_screens.py` を実行して全テスト通過を確認すること。

## テスト

```bash
# 画面遷移テスト（pyxel-mcp 必要、スクリーンショット自動生成）
python tests/test_screens.py

# ズーム機能ユニットテスト（Pyxel不要、ヘッドレス実行可）
python tests/test_zoom.py
```

- `test_screens.py` — pyxel-mcp を使った7つの画面遷移・描画テスト
- `test_zoom.py` — TileZoomAnimation の15項目のユニットテスト（カメラ補間、イージング、状態遷移、tile_image_pos 座標境界）

## blt3d 描画パイプライン

メインボードの描画は **オフスクリーン → blt3d 転送** の統一パイプラインで行われる。ズーム有無を問わず同一コードパスで描画する。

### パイプライン概要

```
BoardView.draw_board_to_image(img)   ← 256x256 イメージバンクに縮小描画
         ↓
TileZoomAnimation.camera_pos/rot/fov ← カメラパラメータ（常時提供）
         ↓
pyxel.blt3d(viewport, img, pos, rot, fov) ← 3D射影でスクリーンに転送
```

### blt3d API（Pyxel 2.x、公式READMEに未記載）

```python
pyxel.blt3d(x, y, w, h, img, pos, rot, fov=None, colkey=None)
```

| パラメータ | 型 | 説明 |
|---|---|---|
| `x, y, w, h` | int | スクリーン上のビューポート矩形 |
| `img` | int | イメージバンク番号（0, 1, 2） |
| `pos` | (x, y, z) | カメラ位置。x,y=イメージ平面上、z=高さ |
| `rot` | (rx, ry, rz) | カメラ回転。rx=傾き角度 |
| `fov` | int | 視野角（度） |
| `colkey` | int | 透過色 |

### カメラパラメータ（経験的に決定）

| 状態 | pos | rot | 備考 |
|---|---|---|---|
| 通常俯瞰 | (128, 220, 120) | (65, 0, 0) | ボード全体が見える |
| ズーム時 | (tile_x, tile_y+20, 25) | (55, 0, 0) | マスに接近 |

**注意点:**
- `rot_x=90`（真下）はほぼ何も映らない。55〜65度が実用的な範囲
- `cam_y` をボードの後方（大きい値）にすると中心に寄る
- イージング: ズームイン `1-(1-p)²`、ズームアウト `1-p²`

### イメージバンク使用状況

| バンク | 用途 |
|---|---|
| 0 | フォント（BDFフォント描画用） |
| 1 | （予備） |
| 2 | **ボードオフスクリーン描画** ← blt3d 転送元 |

### オフスクリーン描画の縮小率

ボード（グリッド 0〜7、40タイル）を 256x256 に収めるため縮小して描画する。

| ビュー | タイルサイズ | ステップ |
|---|---|---|
| TopView | OFF_TILE=20, OFF_GAP=2 | OFF_STEP=22 px/マス |
| Isometric | OFF_ISO_W=30, OFF_ISO_H=15, OFF_ISO_DEPTH=8 | — |

各ビューの `__init__` で `_img_ox`, `_img_oy` を計算してボードを 256x256 の中央に配置する。

### 関連ファイル

| ファイル | 役割 |
|---|---|
| `src/ui/animation.py` → `TileZoomAnimation` | カメラパラメータ管理、イージング補間 |
| `src/views/view_base.py` → `BoardViewBase` | `draw_board_to_image()`, `tile_image_pos()` 基底メソッド |
| `src/views/topview/board_view.py` | TopView のオフスクリーン描画実装 |
| `src/views/isometric/board_view.py` | Isometric のオフスクリーン描画実装 |
| `src/scenes/main_board.py` | 統一描画パイプライン、ズーム状態遷移 |

### main_board.py のズーム状態遷移

```
移動完了 → TILE_ZOOM_IN（20F）→ マスアクション実行 → TILE_ZOOM_OUT（15F）→ 次のプレイヤーへ
```

SubPhase: `TILE_ZOOM_IN`, `TILE_ZOOM_OUT` が追加されている。

## 入力操作ルール

- **キーボード入力は必ず `src/ui/input_helper.py` の `btnp()` / `btn()` を使う。** `pyxel.btnp()` や `pyxel.btn()` を直接呼ばないこと。
- `input_helper` はキーボードとゲームパッド（GAMEPAD1）の入力を統合している。スマホブラウザでは Pyxel の仮想ゲームパッドが表示されるため、この統合レイヤーを使うことでタッチ操作にも対応できる。
- 新しいキー → ゲームパッドのマッピングが必要な場合は `_KEY_TO_GAMEPAD` に追加する。

## コード変更時の注意

- ソースを変更したら `pyxel package . main.py` で `.pyxapp` を再ビルドする
- `.pyxapp` 再ビルド後は `pyxel app2html board-company.pyxapp` で Web 版 HTML も更新する
- ブラウザ実行は GitHub Pages 経由で `board-company.html` を配信（`pyxel app2html` で生成した自己完結型 HTML）
- `tests/test_screens.py` を実行してスクリーンショットが正常に生成されることを確認する
- スクリーンショットが更新された場合は `screenshots/` もコミットに含める

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/munepi0713) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
