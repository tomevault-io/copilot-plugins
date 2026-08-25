## sedori-route

> - この `AGENTS.md` をCodex用の作業ルール正本とする。

# 店舗巡回PWA 作業ルール

## Codex移行メモ

- この `AGENTS.md` をCodex用の作業ルール正本とする。
- `CLAUDE.md` は参照用として残す。重要な差分を見つけたら、削除ではなくこのファイルへ統合する。
- PWA/GAS本番反映は、差分・表示確認・バージョン確認後に進める。

## アプリ概要

店舗せどり向けPWA。店舗選択、選択順ルート、Google Mapsナビ、仕入れ記録、巡回履歴、分析を扱う。

- 配信: GitHub Pages (`daniel-dan-dan/sedori-route`)
- バックエンド: `~/店舗巡回ルート最適化/gas/Code.gs`
- データ: Google スプレッドシート

## PWA内部構造

- `storage.js`: IndexedDBでstores/config/currentRoute/pendingActionsをキャッシュし、同期キューを管理する。
- `api.js`: GAS `doGet`/`doPost` へのfetchラッパー。URLは `localStorage['gas_api_url']` から取得する。
- `route-optimizer.js`: 距離計算、選択順ルート、Google MapsナビURL生成を扱う。
- `router.js`: hashベースSPAルーター。
- `app.js`: 全ビュー描画とイベント処理。既存パターンを優先して変更する。
- `sw.js`: Service Worker。ロゴ/CSS/JSを変更したら必ず `CACHE_NAME` をバンプする。

## GAS API contract

- GET: `getStores`, `getConfig`, `getRouteHistory`, `getRouteStops`, `getRouteAreaVisits`, `getRouteCorrectionSuggestions`, `getPurchases`, `getMemos`, `getFinds`, `getInventoryPurchases`, `recalcPriorityScores`, `updateRouteDate`
- POST: `addStore`, `updateStore`, `deleteStore`, `startRoute`, `updateStop`, `endRoute`, `addStopToRoute`, `addPurchase`, `updateInventoryShop`, `updateConfig`, `updateRouteDate`, `deleteRoute`, `clearHistory`
- POSTは `Content-Type: text/plain` + JSON body。レスポンスがJSONでなくても書き込み自体は成功していることがある。

## 重要ルール

- ユーザーの現在方針は、到着時刻管理より巡回履歴と利益データ蓄積を優先すること。
- ルート作成後は「選択順ルート」を中心に扱い、不要なカードや説明は増やさない。
- PWA変更時は `CACHE_NAME`、cache bust、画面上のバージョン表示、ブラウザ/スマホ反映確認まで行う。
- 店舗削除、履歴削除、GAS本番反映はユーザー確認後に行う。

## ローカル確認

```bash
python3 -m http.server 8000
```

初回アクセス時は、設定画面でGAS Web App URLを入力する必要がある。

---
> Source: [daniel-dan-dan/sedori-route](https://github.com/daniel-dan-dan/sedori-route) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
