## karifol-dot-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # 開発サーバー起動 (http://localhost:3000)
npm run build    # プロダクションビルド
npm run lint     # ESLint実行
```

## Tech Stack

- **Next.js** (App Router) + **TypeScript**
- **Tailwind CSS v4** — `@variant dark (&:where(.dark, .dark *))` でダークモードをクラスベースに設定済み
- **next-themes** — ダーク/ライト切り替え管理
- **react-markdown** — チャット画面でのMarkdownレンダリング

## Architecture

```
app/
├── layout.tsx              # ルートレイアウト。<Providers>でnext-themesをラップ
├── providers.tsx            # "use client" ThemeProvider (next-themes)
├── theme-toggle.tsx         # "use client" ダーク/ライト切り替えボタン
├── site-header.tsx          # karifol ロゴ+サイト名ヘッダー (ホーム以外のページで使用)
├── page.tsx                # トップページ (プロフィール + プロダクト一覧)
├── globals.css             # Tailwind import + ダークモードCSS変数
├── api/
│   └── chat/
│       └── route.ts        # /api/chat プロキシ → 上流 STREAMING_API_URL/ask へPOST転送
└── jobnyaha_ai/
    ├── page.tsx            # じょぶにゃはAI トップ（会話する／育てる の選択画面）
    ├── chat/
    │   └── page.tsx        # チャット画面。SSEストリーミング受信、user_id="trial-user"固定
    └── train/
        └── page.tsx        # 育てる画面（口癖・性格・エピソードの3タブ、現状UIのみ）
public/
├── karifol.png             # プロフィールアイコン
└── jobnyaha.jpg            # じょぶにゃはAIアイコン
```

## Key Patterns

**ダークモード**: `globals.css` でCSS変数を `.dark` クラスで切り替え（メディアクエリではない）。各コンポーネントで `dark:` Tailwindクラスを使う。

**新規ページ**: `app/<slug>/page.tsx` を作成。`ThemeToggle` と `SiteHeader` を忘れずにインポートすること。

**プロダクト追加**: `app/page.tsx` の `products` 配列にエントリを追加する。プロダクトリンクは同タブで開く（`target="_blank"` は使わない）。

**APIプロキシ**: 上流APIのURL・キーは `.env.local` に `NEXT_PUBLIC_` なしで定義し、`app/api/*/route.ts` 経由でアクセスする。クライアントから直接外部APIを叩かない。上流エンドポイントは `/ask`（`route.ts` 参照）。

**チャットのSSE形式**:
```
data: {"type": "content", "text": "..."}
data: {"type": "tool_call", "tool": "..."}
data: {"type": "tool_result"}
data: {"type": "info", "content": "token_limit_reached"}
data: {"type": "error", "error": "monthly_limit_reached"|"token_limit_reached"}
data: {"type": "done"}
```

**環境変数**:
```
STREAMING_API_URL=...   # じょぶにゃはAI バックエンドURL (サーバーサイドのみ)
STREAMING_API_KEY=...   # APIキー (サーバーサイドのみ)
TRAIN_API_URL=...       # 育てるAPI バックエンドURL (サーバーサイドのみ)
TRAIN_API_KEY=...       # APIキー (サーバーサイドのみ)
```

## Amplify デプロイ

**プラットフォーム**: WEB_COMPUTE (Next.js - SSR)

**環境変数の注意点**: Amplify WEB_COMPUTE では、サーバーサイドの環境変数（`NEXT_PUBLIC_` なし）はビルド前に `.env` ファイルへ書き出さないと Lambda に反映されない。Amplifyコンソール → Hosting → ビルドの設定 で以下を設定する：

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm ci --cache .npm --prefer-offline
    build:
      commands:
        - echo "STREAMING_API_URL=$STREAMING_API_URL" >> .env
        - echo "STREAMING_API_KEY=$STREAMING_API_KEY" >> .env
        - echo "TRAIN_API_URL=$TRAIN_API_URL" >> .env
        - echo "TRAIN_API_KEY=$TRAIN_API_KEY" >> .env
        - npm run build
  artifacts:
    baseDirectory: .next
    files:
      - '**/*'
  cache:
    paths:
      - .next/cache/**/*
      - .npm/**/*
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karifol)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/karifol)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
