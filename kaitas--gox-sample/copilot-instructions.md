## gox-sample

> Crypto Lounge GOX（東新宿のクリプトラウンジ）の新ウェブサイト。

# GOX Website - Headless WordPress Project



## プロジェクト概要

Crypto Lounge GOX（東新宿のクリプトラウンジ）の新ウェブサイト。
WordPressからHeadless化し、LINE公式への問い合わせ動線を強化する。

## 技術スタック

- **フレームワーク**: Astro 5.x
- **スタイリング**: Tailwind CSS
- **ホスティング**: Cloudflare Pages（推奨）
- **CMS**: WordPress REST API（イベント情報のみ動的取得）

## ディレクトリ構造

```
gox-site/
├── src/
│   ├── components/     # Astroコンポーネント
│   │   ├── Header.astro
│   │   ├── Hero.astro
│   │   ├── About.astro
│   │   ├── Events.astro      # WP REST API連携
│   │   ├── Price.astro
│   │   ├── RentalCTA.astro   # LINE CTA（重要）
│   │   └── Footer.astro
│   ├── layouts/
│   │   └── Layout.astro      # フローティングLINEボタン含む
│   ├── pages/
│   │   └── index.astro
│   └── styles/
│       └── global.css
├── public/
│   └── favicon.svg
├── astro.config.mjs
├── tailwind.config.mjs
├── tsconfig.json
└── package.json
```

## セットアップ手順

```bash
# 依存関係インストール
npm install

# 開発サーバー起動
npm run dev

# 本番ビルド
npm run build

# ビルド結果プレビュー
npm run preview
```

## デプロイ（Cloudflare Pages）

### 方法1: Wrangler CLI
```bash
npm install -g wrangler
wrangler pages deploy dist --project-name=gox-site
```

### 方法2: Cloudflare Dashboard
1. Cloudflare Dashboard → Pages → Create a project
2. GitHubリポジトリを接続、または `dist/` フォルダを直接アップロード
3. ビルド設定:
   - Build command: `npm run build`
   - Build output directory: `dist`

### 方法3: GitHub Actions（推奨）
```yaml
# .github/workflows/deploy.yml
name: Deploy to Cloudflare Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm install
      - run: npm run build
      - uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: gox-site
          directory: dist
```

## 重要な設定

### WordPress REST API エンドポイント

イベント取得: `https://cryptoloungegox.com/wp-json/wp/v2/event?per_page=6&_embed`

カスタム投稿タイプ `event` が存在する前提。存在しない場合は `posts` に変更するか、WordPress側でREST API公開設定が必要。

### LINE公式アカウント

- URL: `https://lin.ee/2MvznpMl`
- フローティングボタン: `src/layouts/Layout.astro`
- メインCTA: `src/components/RentalCTA.astro`

## カスタマイズポイント

### カラーパレット変更
`tailwind.config.mjs` の `colors` セクションを編集:
```js
colors: {
  'gox-black': '#0a0a0a',
  'gox-gold': '#d4af37',
  'gox-purple': '#8b5cf6',
  'gox-cyan': '#22d3d1',
}
```

### イベント取得数変更
`src/components/Events.astro` の `per_page` パラメータを調整

### 料金更新
`src/components/Price.astro` の配列データを編集

## トラブルシューティング

### イベントが表示されない
1. WordPress REST APIが有効か確認: `https://cryptoloungegox.com/wp-json/wp/v2/event`
2. カスタム投稿タイプ `event` のREST API公開設定を確認
3. CORS設定を確認（必要に応じてWordPress側で許可）

### ビルドエラー
```bash
# キャッシュクリア
rm -rf node_modules .astro dist
npm install
npm run build
```

### Tailwindスタイルが効かない
`tailwind.config.mjs` の `content` パスを確認

## 今後の拡張案

- [ ] イベント詳細ページ（動的ルーティング）
- [ ] ニュース/お知らせセクション追加
- [ ] GOX PRO申し込みフォーム統合
- [ ] 多言語対応（i18n）
- [ ] Google Analytics / Plausible導入

## 連絡先

- 現行サイト: https://cryptoloungegox.com/
- LINE公式: https://lin.ee/2MvznpMl
- Discord: https://discord.gg/X6EbTMbE93

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaitas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kaitas)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
