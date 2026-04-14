## unipic

> このファイルは **Gemini CLI** のためのプロジェクト情報を提供します。

# Gemini.md

このファイルは **Gemini CLI** のためのプロジェクト情報を提供します。

## プロジェクト: UniPic

### 概要
UniPicは、重複画像の検出と削除を行うElectronベースのデスクトップアプリケーションです。バイナリハッシュ比較（MD5）とパーセプチュアルハッシュ（pHash）比較の両方を使用して、同一画像と視覚的に類似した画像を検出します。

### 技術スタック
- **フレームワーク**: Electron v27.0.0
- **言語**: JavaScript (Node.js)
- **画像処理**: Sharp v0.33.5 (高速画像処理ライブラリ)
- **ビルドツール**: electron-builder v24.6.4
- **パッケージマネージャー**: npm

### 主要機能
1. **重複画像検出**
   - バイナリ同一（MD5ハッシュ）検出
   - 視覚的類似（パーセプチュアルハッシュ）検出
   - ハミング距離による類似度計算（0-20の閾値調整可能）

2. **UI/UX**
   - ドラッグ&ドロップによるフォルダ選択
   - リアルタイム進捗表示
   - 画像比較ビューア（フルスクリーン対応）
   - キーボードショートカットサポート

3. **パフォーマンス**
   - 並列処理によるハッシュ計算
   - バッチ処理でメモリ効率を最適化
   - CPUコア数×2のワーカー数で最適化

### アーキテクチャ
- **メインプロセス** (`main.js`): ファイルシステム操作、画像処理、IPC管理
- **レンダラープロセス** (`renderer.js`): UI表示、ユーザーインタラクション
- **画像ビューア** (`image-viewer.js`): 拡大表示と画像比較機能

### 対応画像フォーマット
JPEG, PNG, WebP, AVIF, TIFF, BMP, GIF, HEIC

### 開発コマンド
- `npm start`: 本番モードで起動
- `npm run dev`: 開発モードで起動（DevTools有効）
- `npm run build-mac`: macOS向けビルド（DMG + ZIP）
- `npm run build-win`: Windows向けビルド
- `npm run build-linux`: Linux向けビルド

### 重要な技術的詳細
1. **Sharp依存関係**: Electronでネイティブモジュールを使用するため、ビルド前に`electron-rebuild`が必要
2. **パーセプチュアルハッシュ実装**: 32x32グレースケール → 8x8 DCT → 63ビットハッシュ
3. **IPC通信**: メインプロセスとレンダラープロセス間で画像処理と進捗を通信

### 現在の課題と改善の可能性
- メモリ使用量の最適化（大量画像処理時）
- より高度な重複検出アルゴリズムの実装
- クラウドストレージとの連携
- 機械学習による画像分類機能の追加

### プロジェクトディレクトリ構成
```
UniPic/
├── main.js           # メインプロセス
├── renderer.js       # レンダラープロセス（メインUI）
├── image-viewer.js   # 画像ビューア
├── hash-worker.js    # ハッシュ計算ワーカー
├── index.html        # メインウィンドウHTML
├── image-viewer.html # 画像ビューアHTML
├── icon/             # アプリアイコン
├── dist/             # ビルド出力
└── node_modules/     # 依存関係
```

### Claude Codeとの連携
Claude Codeがこのプロジェクトで作業する際は、CLAUDE.mdに記載された指示に従います。Gemini CLIはWeb検索、技術調査、設計提案などで補助的な役割を果たします。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Junpeiwada)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Junpeiwada)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
