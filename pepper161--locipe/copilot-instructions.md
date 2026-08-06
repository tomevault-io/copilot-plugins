## locipe

> ﻿# Repository Guidelines

﻿# Repository Guidelines

## Project Structure & Module Organization
Next.js App Router files sit in `app/`. Route handlers live in `app/api/gemini`, `app/api/ingredients`, and `app/api/recipes`, while UI pages/components stay under the same route segment. Shared UI primitives live in `components/ui`; composite widgets belong in `components/`. Helpers, Prisma client glue, and shared types live in `lib/`. Prisma schema, migrations, and the curated recipe seed data remain in `prisma/`; static assets (ingredient and dish photos) belong in `public/`. Product and UX notes are collected in `docs/`, notably `docs/app-design.md`.

## MVP Scope & Data Requirements
- 画像アップロード/撮影は常に 1 枚のみを受け付け、複数映っていても主要食材だけを抽出する。
- Gemini 呼び出し時にユーザのヒント（色、似た食材、位置情報）をプロンプトへ追加する。
- 食材の同義語（`okra`、`lady’s finger`、`ガンボ` など）を `prisma` の seed データに保存し、レシピ紐付けを文字列一致で実現する。
- 最低 10 件のレシピを事前登録し、各レシピがヒーロー画像、材料配列、手順、1 人前基準の分量を持つ。
- レシピ検索は常に 4 件の候補をカード表示し、詳細画面は Cookpad 風に構成する。
- 人数切り替えボタンで 1→2→3 人前と分量を自動スケールさせる。

## Build, Test, and Development Commands
- `npm run dev` — Next.js 開発サーバーを起動。
- `npm run build` — 本番ビルドを生成。
- `npm start` — 生成済みビルドを本番モードで提供。
- `npm run lint` — ESLint を実行。
- `npx prisma migrate dev --name <label>` — スキーマ変更を適用しマイグレーションを作成。
- `npx prisma generate` / `npx prisma studio` — Prisma Client の再生成と DB ブラウズ。

## Coding Style & Naming Conventions
TypeScript と React サーバーコンポーネントを基本とし、インタラクティブ要素だけ `"use client"` を付ける。ファイル/コンポーネントは `PascalCase`, フックやユーティリティは `camelCase`, 複合フォルダーは kebab-case。インデントは 2 スペース、Tailwind ユーティリティを `clsx` と `tailwind-merge` で整理する。文言は自然な日本語で route 付近に定義し、PR 前に `npm run lint` を通す。

## Testing Guidelines
テストは lint＋手動確認が前提。最低限: ① 画像アップロード → Gemini 同定 → 4 件候補, ② レシピ選択 → Cookpad 風詳細, ③ 人数切替で分量が比例, ④ 異なる呼称入力 (`lady’s finger`) が既存食材 (`okra`) にマッチ。自動テストを書く場合は対象フォルダー配下に `*.spec.tsx` などを配置し、複雑なフローは `docs/` に Playwright 手順を記録する。

## Commit & Pull Request Guidelines
コミットメッセージは短い命令形（例: `Implement core MVP features`, `prismaのバージョン変えました`）。スキーマ更新時は Prisma マイグレーションや生成物も同梱する。PR では概要、主要変更点、必要なコマンド（例: `npx prisma migrate dev`）、UI 変更のスクリーンショット、関連 Issue を記載する。1 PR = 1 トピックを徹底する。

## Security & Configuration Tips
`.env.example` を `.env` にコピーし、`DATABASE_URL` と `GOOGLE_GEMINI_API_KEY` を設定してから Prisma コマンドを実行する。`.env` や秘匿情報をコミット/PR 説明に含めない。キーが露出した場合は即座にローテーションし、デプロイ環境のシークレットも更新する。

---
> Source: [Pepper161/locipe](https://github.com/Pepper161/locipe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
