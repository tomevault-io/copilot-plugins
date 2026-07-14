## api-billing-platform

> - バックエンド: Go（標準構成、Clean Architecture志向）

# Copilot Instructions: api-billing-platform 開発・テスト・マイグレーション運用指示

---

## 技術スタック・実装方針
- バックエンド: Go（標準構成、Clean Architecture志向）
- フロントエンド: React（Vite構成）
- DB: PostgreSQL
- インフラ: Docker + docker-compose
- マイグレーション: golang-migrate

## DBマイグレーション運用
- マイグレーションファイルは `backend/migrations/` に配置
- golang-migrateを利用し、以下コマンドで適用  
  ```
  docker compose run --rm migrate -database 'postgres://...' -path /app/migrations up
  ```
- 本番用DBとテスト用DB（db_test）は分離運用  
  - テスト用DBはCIやローカルテストで利用
  - マイグレーションは本番・テストDBそれぞれに適用

## テスト運用
- Goテストは以下3パターンで実行可能
  1. ホスト環境: `go test ./...`
  2. Dockerコンテナ: `docker compose run --rm builder go test ./...`
  3. CI（GitHub Actions等）: DBセットアップ→マイグレーション→テスト実行
- テスト用DB（db_test）はテスト前に初期化・マイグレーションを必ず実施
- テストデータ投入は `test_seed.sql` を活用

## docker-compose.yml設計（推奨例）
- `api`: Goアプリ本体
- `db`: 本番用PostgreSQL
- `db_test`: テスト用PostgreSQL
- `builder`: Goビルド用
- `migrate`: golang-migrate実行用
- 必要に応じて `frontend` サービスも追加

## 環境変数管理
- `.env`ファイルで一元管理（DB接続情報、APIキー等）
- docker-compose.ymlでenv_file指定
- CI環境ではGitHub Secrets等で安全に管理

## CI連携
- push/pull request時に自動テスト・マイグレーション検証
- DBセットアップ→マイグレーション→Goテスト→フロントテスト（必要に応じて）

## マイグレーション・テストデータ運用フロー
1. マイグレーションファイル作成（命名規則厳守）
2. 本番/テストDBへマイグレーション適用
3. テスト用DBに初期データ投入（必要ならSQLファイルで）
4. Goテスト実行


## フロントエンド「画面が真っ黒」問題の解消方法
- ルーティング設定（routes.ts等）が正しいか確認（URLとファイル名の一致、小文字統一）
- ページコンポーネントファイル（例: login.tsx, register.tsx）が存在し、export defaultされているか確認
- Vite開発サーバーを一度停止し、再起動（キャッシュ・HMR不具合対策）
- グローバルCSSや親レイアウトの影響で背景・文字色が黒になっていないか確認
- catch-all.tsx等で該当パスが何も返していない場合、明示的にimportする
- ルート定義・ファイル名の大文字/小文字不一致に注意（OS依存で失敗することあり）
- それでも解消しない場合は、エラーメッセージ・コンソールログを確認

## フロントエンド運用ルール追加
- `react-router-dom`は使用禁止（ルーティングは独自実装またはVite/React標準のみ利用）

---

この指示に従えば、開発者が迷わず一貫した運用が可能です。

---
> Source: [nex50792/api-billing-platform](https://github.com/nex50792/api-billing-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
