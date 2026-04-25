## sakana-hen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## コマンド

パッケージマネージャーは **pnpm** を使用する。

```bash
pnpm install       # 依存パッケージのインストール
pnpm run dev       # 開発サーバー起動 (http://localhost:5173/)
pnpm run build     # dist/ へビルド
```

テスト専用コマンドは存在しない。`src/test/` 以下の HTML ファイルをブラウザで直接開いて検証する。

## アーキテクチャ

### 概要

「さかなへん」クイズゲーム。HTML Canvas を使ったシングルページアプリ。Vite でビルドし、ブラウザ上で動作する。

### エントリポイントとゲームフロー

- `src/index.html` — メインページ。`<canvas id="id_canvas">` に描画する。`SakanaHen` クラスをインスタンス化して起動。
- `src/js/index.ts` — `SakanaHen` を re-export するだけ。
- `src/js/sakana.ts` — ゲーム本体。`SakanaHen` クラスが全ゲームロジックを担う。

ゲームの状態遷移: **タイトル画面** → `startGame(level)` → **クイズ進行**（`timer()` がフレームループ） → `endGame()` → **結果画面** → タイトルへ戻る

### ディレクトリ構成

```
src/
  index.html         メインページ
  js/
    index.ts         SakanaHen の re-export
    sakana.ts        ゲーム本体 (SakanaHen クラス)
    consts/
      dakana_data.ts  魚偏クイズデータ (LST_SAKANA_HEN: [漢字, 読み] の配列)
      def_data.ts     画像・音声リスト定義、結果メッセージ、フォント定数
    libs/
      easing.ts       イージング関数 (easeOutElastic 等)
      log.ts          Log クラス (debug フラグで console.log を制御)
    types/
      rectangle.ts    Rectangle 型定義
  assets/
    images/           SVG・PNG 画像
    sounds/           OGG・MP3 音声ファイル
  test/               サウンド再生テスト HTML 等
dist/                 ビルド出力先
```

### 重要な設計ポイント

- **座標補正**: Canvas が CSS で拡大縮小される場合に `vHeight` を使って仮想座標へ変換する (`getVPos()`)。
- **画像・音声の読み込み**: `import.meta.url` を使って Vite のアセット解決を利用する。音声は OGG 優先、非対応なら MP3。
- **難易度設定**: `sakana.ts` 内の `LEVEL_INFO` 配列でレベルごとの制限時間・問題数・選択肢数を変更できる。
- **クイズデータのカスタマイズ**: `src/js/consts/dakana_data.ts` の `LST_SAKANA_HEN` を編集する。
- **環境変数**: `vite.config.mjs` のカスタムプラグインが `%=VITE_XXX%` 形式のテンプレートを HTML 内で展開する (OG 画像 URL 等)。
- **デバッグログ**: `Log.info()` は開発モード (`import.meta.env.DEV`) のときのみ出力される。
- **フレームレート**: `FRAME_RATE = 90` (fps)。スマホで重い場合はここを下げる。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tghkuma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
