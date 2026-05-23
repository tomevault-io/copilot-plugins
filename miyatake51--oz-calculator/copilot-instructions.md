## oz-calculator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト現状

このリポジトリは実装着手前の状態。コードはまだ存在せず、以下のドキュメントが揃っている:

- `README.md` — 一行のプロジェクト名・概要のみ
- `concept_2026-05-10.md` — 概念探索フェーズの成果物
- `plan_2026-05-10.md` — 実装設計書（プラン B 確定版）。ファイル構成、データ設計、実装ステップ、テスト項目、リスク対策まで網羅済み。実装着手時はこのファイルに従って進める
- `reference/visual-reference-heddon-1904.jpeg` — ビジュアル基準画像（1904年 Heddon "Dowagiac Expert" カタログ）

## ドメインと目的

バスフィッシング用ルアーの oz↔g を瞬時変換する単一ユーザー向け PWA。対象 oz 範囲は 1/32 oz 〜 4 oz 帯（実流通する21種）。主用途は (a) 店頭・通販での購入判断、(b) 自宅でのタックル準備時のロッド適合確認。

成功条件: 起動から目的の oz 値到達まで「3 タップ以内」、購入後 3 ヶ月時点でも週 1 回以上の起動継続。

主動機は「所有満足 + Claude Code 実践題材 + 実用ツール」の三位一体。UI/ビジュアル設計の質を軽視しない。

## 確定済み技術スタック

`plan_2026-05-10.md` で以下が確定済み:

- ビルド: Vite + Vanilla TypeScript（フレームワーク不使用）
- PWA: vite-plugin-pwa（Cache First、`registerType: 'autoUpdate'` + skipWaiting）
- デプロイ: Cloudflare Pages（ビルド `npm run build` / 出力 `dist` / `NODE_VERSION=20`）
- フォント: `@fontsource` で self-host（Google Fonts 直リンク不使用、オフライン要件のため）
- 状態管理: モジュールレベル変数 + CustomEvent（ライブラリ不使用）
- ドメイン: `*.pages.dev` で運用（独自ドメインなし）

## 確定済み仕様（実装着手前の論点はすべて決着）

`concept_2026-05-10.md` の「残存不確実性」6項目はすべて plan B で決着済み:

1. カテゴリ分類 → 5分類（極小 / ライト / ミディアム / ヘビー / ビッグベイト）
2. g 表示精度 → 小数第 1 位で固定（`Math.round(oz * 28.3495 * 10) / 10`）
3. 逆変換（g→oz） → 単位トグルで双方向対応
4. ビジュアル方針 → 1904年 Heddon カタログ基準（クリーム背景・ダークインク・セリフ書体・紙質感）
5. オフライン動作 → Service Worker で完全オフライン化
6. データ拡張 → 典型ルアー例 2〜3 件を各カードに併記

実装着手時に requirements-plan スキルで再ヒアリングする必要はない。

## 実装着手時の進め方

ユーザーから「実装して」「進めて」等の指示があった場合:

1. `plan_2026-05-10.md` を読み、セッション 1 のステップ 1-1 から順に進める
2. 2 セッション分割が前提（セッション1: 機能完成 4〜5h、セッション2: ビジュアル+デプロイ 2〜3h）
3. ビジュアル実装時は `reference/visual-reference-heddon-1904.jpeg` を必ず参照（色・タイポ・装飾の根拠画像）
4. 設計書に修正・追加が発生したら `plan_2026-05-10.md` を直接更新する

## ビルド・テスト・lint コマンド

実装着手前のため未確立。セッション 1-1（Vite 初期化）完了時点で以下が利用可能になる予定:

- 開発サーバ: `npm run dev`
- 本番ビルド: `npm run build`
- 型チェック: `npx tsc --noEmit`
- プレビュー: `npm run preview`

これらが動作するようになった時点で、本セクションを実コマンド・実行例に書き換えること。

## ファイル構成（plan B 確定）

実装着手時に作るディレクトリ・ファイルは `plan_2026-05-10.md` の「変更対象ファイル」表が正本。ハイレベルな構造:

- `src/data/lures.json` — 21 件のルアーデータ（ozLabel / ozValue / grams / category / examples）
- `src/types/lure.ts` — `Category` / `Lure` 型
- `src/utils/{convert,format}.ts` — 換算・表示整形
- `src/state.ts` — `mode: 'oz'|'g'` と `category` をモジュール変数で保持、`state:change` CustomEvent で通知
- `src/components/{Header,Tabs,LureCard}.ts` — DOM 直接操作の素朴なコンポーネント
- `src/styles/{tokens,base,components}.css` — トークンは `plan_2026-05-10.md` の CSS 変数定義が正本
- `src/pwa/install-prompt.ts` — iOS ホーム追加案内（`navigator.standalone` 判定）

## リスク・注意点（最重要）

`plan_2026-05-10.md` の「リスク・注意点」セクションが正本。特に以下は実装中に常に意識する:

1. Service Worker × Cloudflare CDN の二重キャッシュ — `index.html` と SW 本体は `_headers` で `Cache-Control: no-cache` 明示
2. JSON 手編集ミスで起動不能 — 起動時 `try/catch` でフォールバック表示、簡易バリデータ実装
3. iOS PWA 落とし穴 — `viewport-fit=cover` + `env(safe-area-inset-*)` の漏れ厳禁、`color-scheme: light only` で OS ダークモード時のズレ回避

## 作業の進め方

- 既に plan B が確定しているため、新規実装依頼時に requirements-plan スキルで再ヒアリングはしない（不確実性は決着済み）
- ただし plan に書かれていない新機能要望が来た場合は、現スコープ外として扱うか plan を更新してから着手するかをユーザーに確認する
- ローカル開発サーバ起動・デプロイ操作は Claude Code が代行する前提（ユーザーはプログラミング初心者）
- 危険・不可逆な操作（git push、ファイル削除、依存削除など）の前は必ず日本語で説明してから確認を取る

---
> Source: [miyatake51/Oz-Calculator](https://github.com/miyatake51/Oz-Calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
