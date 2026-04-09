## ai-chatapp

> ai-chatapp プロジェクトの構造ガイド

# ai-chatapp プロジェクト構造ガイド

このルールはリポジトリ全体の構造を示す早見表です。  
AI が各ファイルの役割を素早く理解できるよう、主要ディレクトリと代表的なファイルを一覧化しています。  
ファイルリンクは `[file](mdc:path/to/file)` 形式で記述しています。

## 主要ディレクトリ

- `chatapp/` — Next.js アプリ本体  
  - `firebase.ts` — Firebase 設定  
  - [next.config.mjs](mdc:chatapp/next.config.mjs) — Next.js 設定  
  - `src/app/` — Next.js App Router  
    - [layout.tsx](mdc:chatapp/src/app/layout.tsx) — ルートレイアウト  
    - [page.tsx](mdc:chatapp/src/app/page.tsx) — トップページ  
    - `api/` — API ルート  
      - [openai/route.ts](mdc:chatapp/src/app/api/openai/route.ts)  
      - [claude/route.ts](mdc:chatapp/src/app/api/claude/route.ts)  
      - その他 `/documents/`, `/rooms/` などの機能別エンドポイント  
    - `auth/` — 認証ページ (`login`, `register`)  
    - `components/` — UI コンポーネント群  
  - `src/lib/` — サーバーサイドユーティリティ  
    - [db.ts](mdc:chatapp/src/lib/db.ts) — Drizzle ORM + PostgreSQL 接続  
  - `src/constants/` — 定数群  
    - [models.ts](mdc:chatapp/src/constants/models.ts) — 利用可能な AI モデル一覧  

- ルート直下  
  - [docker-compose.yml](mdc:docker-compose.yml) — Postgres などのサービス定義  
  - [init-db.sql](mdc:init-db.sql) — DB スキーマ初期化  
  - [README.md](mdc:README.md) — プロジェクト概要  

## 主要技術スタック

- Next.js 14 (App Router)
- React 18 + TypeScript
- Tailwind CSS
- Drizzle ORM + PostgreSQL
- OpenAI / Claude API 連携

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/y16ra)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/y16ra)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
