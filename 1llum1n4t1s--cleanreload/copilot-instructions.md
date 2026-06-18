## cleanreload

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

Clean Reload — ワンクリックでキャッシュ完全クリア+リロード（スーパーリロード）を実行するChrome拡張機能（Manifest V3）。

リロード時の処理（すべて fire-and-forget・origin スコープ）:
1. Service Worker 登録を削除（`chrome.browsingData.remove({origins}, {serviceWorkers: true})`）
2. CacheStorage を削除（`chrome.browsingData.remove({origins}, {cacheStorage: true})`）
3. HTTP キャッシュを削除（`chrome.browsingData.removeCache({origins})`）
4. HTTP キャッシュをバイパスしてリロード（`chrome.tabs.reload({ bypassCache: true })`）

## ビルドコマンド

```bash
npm install                # 依存関係インストール（sharp + puppeteer/Chromium DL、初回は重い）
npm run generate-icons     # icons/icon.svg → icons/icon-{16,48,128}.png（sharp のみ使用）
npm run generate-screenshots # webstore/*.html → webstore/images/*.png（puppeteer 使用）
npm run build              # 上記2つを順次実行
```

- テスト・Lint・フォーマッタは**未定義**。`npm test` などは存在しない。
- 拡張機能本体の開発には `sharp` しか不要。`generate-screenshots` はストア画像用途で、普段のコード編集では実行しなくてよい。

## 開発時の読み込み

1. `npm install && npm run generate-icons` でアイコン PNG を生成（`icons/icon-*.png` は gitignore なので初回必須）
2. Chrome で `chrome://extensions/` → デベロッパーモードON → 「パッケージ化されていない拡張機能を読み込む」→ リポジトリルートを選択

## パッケージング

```powershell
.\zip.ps1                  # Windows
./zip.sh                   # Linux/macOS
```
`clean-reload.zip` に `manifest.json` + `icons/` + `src/` のみ含める（`node_modules`, `webstore/`, `scripts/` は除外）。

## リリース (Chrome Web Store 自動公開)

1. `manifest.json` の `version` を上げてコミット
2. `release/x.y.z`（`x.y.z` は manifest の version と一致必須）ブランチを push
3. [.github/workflows/publish.yml](.github/workflows/publish.yml) が起動 → `npm ci` → `generate-icons` → zip → `chrome-webstore-upload-cli --auto-publish`
4. ブランチ名と manifest バージョンの整合性チェックあり（不一致なら fail）

## ディレクトリ構造（統一規約）

```
CleanReload/
├── manifest.json            # ルート直置き
├── icons/                   # アイコン（icon.svg + 生成PNG、PNGはgitignore）
├── src/
│   └── background/
│       └── background.js    # service worker
├── scripts/
│   └── generate-icons.js    # PNGアイコン生成（sharp）
├── webstore/                # ストア申請アセット
│   ├── *.html               # 掲載画像テンプレート
│   ├── generate-screenshots.js  # HTML→PNG変換（puppeteer、scripts/ ではなくここ）
│   └── store-listing.txt    # ストア説明文
├── docs/privacy-policy.md   # ストア申請に必須のプライバシーポリシー
└── .github/workflows/publish.yml  # 自動公開ワークフロー
```

## アーキテクチャ

- **manifest.json** — 拡張機能の定義。権限は `activeTab` + `browsingData`。popupなし（アイコンクリックで即実行）。
- **src/background/background.js** — 唯一のランタイムコード。`chrome.action.onClicked` で、まず `chrome.browsingData.remove({origins}, {cacheStorage, serviceWorkers})` を **await** して SW 登録と CacheStorage の削除を確実に待機（SW が fetch を横取りする race を排除）、続けて HTTP キャッシュ削除と `tabs.reload({bypassCache: true})` を fire-and-forget で発火。ページ注入を使わないため Chrome Web Store 等の制限ページでも同じ挙動で動く。
- **icons/icon.svg** — マスターアイコン。ここを変更すれば `generate-icons.js` で全サイズ生成。
- **webstore/*.html** — ストア掲載画像のHTMLテンプレート。`generate-screenshots.js`（Puppeteer）でPNGに変換。

## 制約事項

- 内部・opaque origin ページ（`chrome:`, `edge:`, `chrome-extension:`, `about:`, `data:`, `javascript:`, `blob:`, `file:`）は `BLOCKED_PROTOCOLS` で早期 return。`new URL()` の throw も try-catch で吸収
- SW/CacheStorage 削除は `await` で完了を待つ（`bypassCache` は HTTP キャッシュ層のみバイパスし SW の fetch 介入はバイパスできないため、古い SW の race を防ぐ必要がある）
- HTTP キャッシュ削除と `tabs.reload({bypassCache:true})` は `.catch()` でログするだけの fire-and-forget（初版から継続する設計判断）

---
> Source: [1llum1n4t1s/CleanReload](https://github.com/1llum1n4t1s/CleanReload) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
