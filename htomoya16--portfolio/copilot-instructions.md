## portfolio

> このリポジトリは、エンジニア向けポートフォリオサイトです。新卒採用担当者とエンジニア面接官の両方に、信頼感・成長意欲・技術的な厚みが伝わる状態を保ちます。

# Portfolio Agent Guide

このリポジトリは、エンジニア向けポートフォリオサイトです。新卒採用担当者とエンジニア面接官の両方に、信頼感・成長意欲・技術的な厚みが伝わる状態を保ちます。

## Project Goal

- 現在の情報設計を活かしつつ、就活向けの可読性と実装品質を優先する
- Bold Experimental な表現を使うが、読みやすさ・保守性・アクセシビリティを壊さない
- Vercel にデプロイでき、`build` が安定して通る状態を維持する

## Tech Stack

- Next.js App Router
- TypeScript
- Tailwind CSS v4
- shadcn style components
- Base UI / GSAP / Anime.js / Lenis / Embla Carousel

## Directory Policy

- App Router のページは `src/app/` に置く
- トップページ section は `src/components/sections/home/` に置く
- layout は `src/components/layout/`、animation は `src/components/animation/` に置く
- 再利用 UI は `src/components/ui/` に置く
- React に依存する再利用 hook は `src/hooks/` に置く
- React に依存しない純粋関数は `src/lib/` に置く
- skills, timeline, projects, links などの表示データは `src/content/` 配下の型付きデータに寄せる
- ページ固有の複雑な CSS を `src/app/globals.css` に増やしすぎない

## Design Policy

- 既製テンプレート感を避け、タイポグラフィ・余白・グリッド・画像の使い方で差を出す
- 研究 / Haptics / Backend の実績が伝わる実画像や生成素材を優先する
- アニメーションは視線誘導、状態変化、モーダル、スクロール体験に意味がある場合だけ使う
- `prefers-reduced-motion` を尊重する
- GitHub / Qiita / Mail への導線は維持する

## Quality Commands

変更後は可能な限り次を確認する。

```bash
pnpm lint
pnpm typecheck
pnpm build
```

軽量な確認は `pnpm check` を使う。

## Library Policy

- 採用・検討中のライブラリは [docs/libraries.md](./docs/libraries.md) にまとめる
- 新しいライブラリを追加する場合は、目的・使いどころ・既存ライブラリとの重複を追記する
- ドット風フォントや短い mono label の文字演出は Anime.js `scrambleText()` を優先し、GSAP ScrambleTextPlugin は使わない

## Docs Policy

- `AGENTS.md` はこのリポジトリで守るべき最小ルールに限定する
- 背景説明、判断履歴、詳細な開発手順は `docs/` に置く
- 実装前の設計判断や刷新方針は `docs/plans/` に置く

---
> Source: [htomoya16/portfolio](https://github.com/htomoya16/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
