## wisteria-io

> このプロジェクトは Astro + Tailwind CSS で構築されたポートフォリオサイトです。

# CLAUDE.md

このプロジェクトは Astro + Tailwind CSS で構築されたポートフォリオサイトです。

## 構成

- `src/components/` — 各セクションのコンポーネント（Hero, About, Skills, Projects, Experience, Links, Nav）
- `src/layouts/Layout.astro` — 共通レイアウト
- `src/styles/global.css` — TailwindCSS + カスタムテーマ
- `serve.cjs` — 静的ファイル配信サーバー
- `.github/workflows/deploy.yml` — CI/CDパイプライン

## ホスティング

- Mac mini でセルフホスト
- Cloudflare Tunnel 経由で公開（ポート開放不要）
- launchd でサーバー・Tunnel を常駐化

## ビルド

```bash
npm ci
npx astro build
```

成果物は `dist/` に出力され、デプロイ時に `/Users/wisteria/portfolio-dist` にコピーされます。

## 注意

- `astro.config.mjs` の `vite.server.allowedHosts` に `wisteria-io.com` が必要
- `devToolbar` は無効化済み

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wisteriahuman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wisteriahuman)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
