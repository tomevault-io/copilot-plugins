## surinuke-gb

> Game Boy / Game Boy Color 用のレーン制回避アクションゲーム。GBDK-2020 v4.3.0 + C言語。

# surinuke-gb 開発者向けドキュメント

Game Boy / Game Boy Color 用のレーン制回避アクションゲーム。GBDK-2020 v4.3.0 + C言語。

## プロジェクト構造

```
surinuke-gb/
├── main.c          # ゲーム全体（単一ファイル）
├── Makefile        # ビルド設定（lcc）
├── Dockerfile      # Docker環境（GBDK 4.3.0）
├── docs/           # 仕様書
└── .github/        # CI/CD（build.yml + release.yml）
```

## ビルド

```bash
make          # ビルド（surinuke-gb.gb を生成）
make clean    # クリーンアップ
```

## ゲームデザイン

### レーン制

- 8レーン構成（タイル列6〜13、壁はタイル列5と14）
- 各レーン幅 = 8px = 1タイル
- プレイヤーはキー押下中に滑らかにピクセル移動し、キーを離すと最寄りのレーン中心にスナップ
- 衝突判定: 敵と同じタイル行 かつ 同じレーン = ゲームオーバー

### 敵の管理（リングバッファ）

- 6体の敵スプライト（sprite ID 1〜6）が常時存在
- 各敵は1行に1体、ランダムなレーン(0〜7)に配置
- 画面下端を通過 → スコア+1 → 最も上にいる敵の1行上にリサイクル＋新しいレーンを割り当て

### 難易度

- 落下速度のみ変化（4段階: 1→2→3→4 px/frame）
- スコア閾値: 20 / 50 / 100

### タイトル画面

- 「SURINUKE」+「Press START」を表示
- START待ちの間にDIV_REGでLFSRシードを撹拌（毎回異なるパターン）

## 技術メモ

### スプライトデータ形式
- 各行は2バイト: [低ビットプレーン, 高ビットプレーン]
- 両プレーンが同値 → 色0（透明）と色3のみ使用
- GBCパレットは4色フル定義が必要（色3が実際の表示色）

### 座標系
- スプライトX: 8 = 画面左端、Y: 16 = 画面上端
- 背景タイル: 列N = 画面ピクセル N×8
- FIRST_LANE_X = 56（タイル列6の左端）、LAST_LANE_X = 112（タイル列13）
- PLAYER_Y = 144（画面下部固定）、PLAYER_ROW = 18（= 144/8）

### 壁タイル
- タイルインデックス128（WALL_TILE_ID）を使用
- GBDKのフォント（printf用）がタイル0〜127を占有するため、128以降を使うことで衝突を回避

### 乱数
- 16-bit LFSR（Linear Feedback Shift Register）で周期65535の疑似乱数を生成
- タップ位置: bit 16, 15, 13, 4（多項式 x^16 + x^15 + x^13 + x^4 + 1、最大長保証）
- DIV_REG（ハードウェアタイマー）でタイトル画面中にシードにエントロピーを追加
- ゲームリスタート時にシードを再撹拌
- 1回の呼び出しで8ビットシフトし、下位8ビットを返す

### GBC判定
- `_cpu == CGB_TYPE` でGBCかDMGかを判定
- DMGではモノクロで動作（パレット設定をスキップ）

## CI/CD

- `build.yml`: プッシュ時に自動ビルド。成果物はArtifacts
- `release.yml`: `v*` タグのプッシュで自動GitHub Release（ROMを添付）

---
> Source: [kako-jun/surinuke-gb](https://github.com/kako-jun/surinuke-gb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
