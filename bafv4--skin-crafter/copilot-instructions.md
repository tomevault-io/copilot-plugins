## skin-crafter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

Skin Crafter はブラウザベースの Minecraft スキンエディタ。React SPA として構築されており、64x64 ピクセルのキャンバス上でレイヤー/グループシステムを使って描画し、リアルタイム 3D プレビューで確認できる。UIおよびコード内コメントは日本語。

## コマンド

- `pnpm dev` — 開発サーバー起動
- `pnpm build` — プロダクションビルド（React Router + Vite）
- `pnpm typecheck` — `react-router typegen` 実行後に `tsc`
- `pnpm start` — プロダクションビルドの配信

テストランナーは未設定。変更の検証には `pnpm typecheck` を使用する。

## アーキテクチャ

**フレームワーク:** React Router v7（SPAモード、`ssr: false`）+ Vite + Tailwind CSS v4。

**状態管理:** 単一の Zustand ストア（[app/stores/editorStore.ts](app/stores/editorStore.ts)）がエディタの全状態を保持する。レイヤー、グループ、ツール、カラー、履歴（差分ベースの undo/redo）、パレット、3Dプレビュー設定など。

**PixelEngine（Web Worker）:** 重いピクセル操作（合成、ピクセル単位の読み書き）は Web Worker で実行し、UI のレスポンスを維持する。メインスレッド側 API は [app/lib/pixelEngine/index.ts](app/lib/pixelEngine/index.ts)（`getPixelEngine()` でシングルトン取得）、メッセージ型定義は [messages.ts](app/lib/pixelEngine/messages.ts)、Worker 本体は [worker.ts](app/lib/pixelEngine/worker.ts)。通信は `postMessage` によるコマンド/レスポンス方式で、`Uint8ClampedArray` バッファを使用する。ストア側にも並行して `LayerPixels`（`RGBA | null` の2次元配列）を保持し、undo/redo スナップショットおよびシリアライズに使用する。

**スキンデータモデル（[app/types/editor.ts](app/types/editor.ts)）:** 64x64 のスキンは `SkinRegion` エントリ（UV座標付き）に分割される。レイヤー（`Layer`）はグループ（`LayerGroup`）に属し、`singleColor`（ベースカラー＋ノイズ）と `direct`（任意のピクセル色）の2モードをサポート。Steve/Alex モデルの違いは腕の幅のみ（4px vs 3px）。

**レンダリングパイプライン:**
- 2D キャンバス（[Canvas2D.tsx](app/components/editor/Canvas2D.tsx)）— Worker からの合成 `ImageData` を描画し、グリッド/オーバーレイ/ハイライトを [skinRenderer.ts](app/lib/skinRenderer.ts) で重ねる
- 3D プレビュー（[Preview3D.tsx](app/components/editor/Preview3D.tsx)）— React Three Fiber を使用。初期バンドルサイズ削減のため遅延読み込み

**UIコンポーネント:** shadcn/ui（new-york スタイル）を [app/components/ui/](app/components/ui/) に配置。エディタ固有のコンポーネントは [app/components/editor/](app/components/editor/)。

**パスエイリアス:** `@components/*` → `./app/components/*`、`@lib/*` → `./app/lib/*`（tsconfig paths）。

## デプロイ

Vercel にデプロイ。GitHub の main ブランチにプッシュすると自動でデプロイされる。

---
> Source: [bafv4/skin-crafter](https://github.com/bafv4/skin-crafter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
