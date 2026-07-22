## animationlinechart

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

D3.jsを使用したアニメーション機能付きラインチャートライブラリ。リサイズ対応、軸の動的変更、カスタムラベル機能を含む。

## 開発コマンド

```bash
# 開発サーバー起動 (ホットリロード対応)
npm run dev

# プロダクションビルド
npm run build

# ビルド結果のプレビュー
npm run preview

# GitHub Pagesデプロイ
npm run deploy
```

## アーキテクチャ

### ファイル構成
- `src/animationLineChart.js`: メインライブラリクラス (ES6モジュール)
- `src/index.js`: デモアプリケーションのエントリーポイント
- `src/index.html`: デモページ
- `src/index.scss`: スタイルシート
- `public/data/example.csv`: サンプルデータ
- `vite.config.js`: Vite設定 (srcをルートディレクトリに設定)

### 技術スタック
- **D3.js v7**: データビジュアライゼーション
- **Vite**: ビルドツール・開発サーバー
- **Sass**: CSSプリプロセッサ
- **Legacy Plugin**: 古いブラウザ対応

### AnimationLineChartクラス
コアクラスは以下の機能を提供:
- **データ読み込み**: CSV形式のデータを動的読み込み
- **アニメーション**: ライン描画のアニメーション (`animateLine()`)
- **軸の動的変更**: X/Y軸の範囲をアニメーション付きで変更 (`setAxisRanges()`)
- **カスタムラベル**: 動的ラベル追加・削除 (`addLabel()`, `removeLabel()`)
- **リサイズ対応**: ResizeObserverによる自動リサイズ
- **状態管理**: 表示ライン状態の管理 (`visibleLines Set`)

### データ形式
CSV形式: `year,key,value`
- `key`は`series`オプションで定義したキーと一致する必要がある
- `series`配列でラインの色・ラベルを定義

## 開発時の注意点

- メインクラスは`src/animationLineChart.js`に集約されている
- デモアプリケーションで各機能の使用例を確認可能
- リサイズ処理やメモリリークを避けるため適切なクリーンアップが実装済み
- D3.jsのスケール機能を活用した軸の動的変更システム

---
> Source: [shimizu/animationLineChart](https://github.com/shimizu/animationLineChart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
