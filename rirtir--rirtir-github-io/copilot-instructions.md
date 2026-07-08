## rirtir-github-io

> このリポジトリで作業する Claude Code / AI エージェント向けのガイドです。

# CLAUDE.md

このリポジトリで作業する Claude Code / AI エージェント向けのガイドです。

## プロジェクト概要

`rirtir.com`（GitHub Pages でホスト）で公開している個人サイト。ゲーム・画像処理・音声処理・ユーティリティなど、各種ブラウザツールを集めたポータルサイトです。

- **ホスト**: GitHub Pages（リポジトリ名 `rirtir.github.io`、独自ドメイン `rirtir.com` を `CNAME` で設定）
- **公開の仕組み**: フォルダを追加して中に `index.html` を置くと `https://rirtir.com/フォルダ名/` で公開される（ビルド工程なし・静的配信）
- **技術構成**: 素の HTML / CSS / JavaScript が中心。ビルドツールやパッケージマネージャは使っていない
- 一部のツールはクライアント完結（オフライン処理）を売りにしている

## ディレクトリ構成

```
/                       ← ポータル（メインページ）
  index.html            ポータルのHTML。中身はほぼ空で links.js が動的生成する
  links.js              ★ ポータルに並ぶ全ツールのカタログ（data配列）＋描画ロジック
  style.css             ポータル専用のスタイル（ダークテーマ）
  CNAME                 独自ドメイン設定（rirtir.com）
  README.md             作者本人の作業メモ・TODO（時系列の日記形式）

  css/                  ★ 全ページ共通のスタイル
    common-style.css      見た目の土台（ボタン・背景色など全サイト共通）
    common-style2.css     特別なUIを持たない一般的なページ用の追加スタイル
    clearEffect_01.css / gameoverEffect_01.css   ゲーム用エフェクト
  js/                   共通スクリプト（clearEffect_01.js / gameoverEffect_01.js）
  assets/               共通アセット（icons/ … SVG・PNGアイコン、img/ … 画像）
  ffmpeg/               ffmpeg.wasm（ffmpeg-core.js / .wasm 等）。音声/動画系ツールが利用
  template/             各種UIのテンプレート・実験用HTML断片（本番配信ではない）

  Game/                 ゲーム（ito, bob, BrainVita, puzzle, BlockPuzzle,
                        AirHockey, Nonogram, incremental, ShelteredGirl など）
  Image/                画像処理ツール（AlphaTool, Resize, PixelArtMaker,
                        ImageEffectLayerTool, MosaicMask, Video2Image など）
  Audio/                音声処理ツール（Video2Audio, AudioInfo,
                        RealTimeAudioAnalyzer, RealTimeWhisper）
  Utility/              その他ツール（PDFJoiner, GohanRoulette,
                        RoundRobinString, jsonl2graph, DepthPreview など）
```

各ツールは「ジャンルフォルダ / ツール名フォルダ / `index.html`」という構造。1ツール = 1フォルダが原則です。

## 主要ファイル

- **`links.js`** … ポータルの内容そのもの。先頭の `data` 配列がカテゴリ（ゲーム / 画像処理 / 音声処理 / ユーティリティ / リンク / おすすめサイト）ごとのリンク一覧。各エントリは `{ title, url, desc, tags, main }`。
  - `main: true` … トップの「主要」フィルタに表示。`false` は「すべて」でのみ表示
  - 新しいツールを追加したら、この `data` 配列に手動で追記する必要がある
- **`index.html`（ルート）** … `main#container` の中を `links.js` が動的描画。「主要 / すべて」トグルと検索ボックスがある

## 共通スタイルの使い方

各ツールの `index.html` からは相対パスで共通CSSを読み込む（ジャンルフォルダ配下＝2階層下なので）:

```html
<link rel="stylesheet" href="./../../css/common-style.css">
<link rel="stylesheet" href="./../../css/common-style2.css">   <!-- 一般的なページのみ -->
```

共通アイコン等も同様に `../../assets/icons/...` で参照する。

- `common-style.css` … ボタン・背景色など**見た目の土台**。多くのページが読み込む
- `common-style2.css` … 特別なUIを持たない**一般的なページ向け**の追加分。これだけでUIが完成する場合もある
- 独自UIの強いページ（多くのゲーム）は `common-style.css` のみ、または個別CSSを併用

新規ページを作るときは、まず共通CSSに乗せられないか検討する（作者はスタイル統一を重視している）。

## サーバーを使うページ（Render 実装）

**サーバーが必要なタイプのページは、このリポジトリの外にある。** ワークスペースに登録された兄弟フォルダ `../Renderを使った実装/`（`rirtir.github.io.code-workspace` に登録済み）に置かれています。

- FastAPI + uvicorn + WebSocket による Python サーバー。[Render](https://render.com/) に無料デプロイして運用
- 例: `ito_online/`（`server.py` + `game.py` + `requirements.txt` + `static/`）、`bahamut_online/`、`render_test/` など
- 公開URLは `https://<アプリ名>.onrender.com/` 形式。ポータル（`links.js`）からはこの外部URLでリンクしている（例: `ito-online.onrender.com`）
- `requirements.txt` に依存を記述。ローカル起動は `uvicorn server:app`（多くは `host="0.0.0.0", port=10000`）
- **このリポジトリ内には該当コードは無い**ので、サーバー実装の話が出たら `../Renderを使った実装/` を見ること

## 開発時の注意

- **ビルド不要**: `index.html` をブラウザで直接開く / ローカルサーバーで配信すれば確認できる。デプロイは `main` への push（GitHub Pages が自動反映）
- **`index2.html` / `index3.html`**: 一部フォルダにある。安定版を `index.html`、調整中のものを `index2.html` 等として残す運用（例: `Image/EditSVG/`）。`index.html` が本番
- **README.md は作者の作業日記**。未解決のバグ・TODO が時系列で書かれている（例: PDF結合ツールの回転バグ、カラーパレット抽出の不具合など）。現状把握に有用
- **ライセンス表記**: ffmpeg.wasm（LGPL/GPL）や利用アイコンのクレジットが README 末尾にある。配布物を触るときは維持すること
- コミットメッセージ・コメント・UI文言は日本語が基本

---
> Source: [rirtir/rirtir.github.io](https://github.com/rirtir/rirtir.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
