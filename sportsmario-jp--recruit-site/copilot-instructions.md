## recruit-site

> このファイルは Claude Code が自動的に読み込むプロジェクト文脈です。

# スポーツマリオ 採用サイト プロジェクトガイド

このファイルは Claude Code が自動的に読み込むプロジェクト文脈です。
全担当者が共通の前提で作業できるようにここに集約しています。

## サイト概要

- **公開URL**: https://recruit.sportsmario.co.jp
- **公開先**: AWS Amplify Hosting
- **作業ディレクトリ**: `C:\recruit-mock`
- **採用区分**: 新卒採用 / 中途キャリア / アルバイト・パート / SMSA（インフルエンサー採用）

## デプロイ方式（最重要）

```
ローカル編集
   ↓
git push codecommit main  ← 本番反映（必須）
git push origin main      ← GitHubバックアップ（任意だが推奨）
   ↓
AWS Amplify が自動ビルド → デプロイ（約2〜5分）
   ↓
recruit.sportsmario.co.jp に反映
```

⚠️ **`git push codecommit main` を忘れると本番に反映されません。** GitHubだけのpushは無効です。

## Gitリモート

| 名前 | URL | 用途 |
|---|---|---|
| `codecommit` | `https://git-codecommit.ap-northeast-1.amazonaws.com/v1/repos/recruit-site` | 本番デプロイ元（必須） |
| `origin` | `https://github.com/sportsmario-jp/recruit-site` | バックアップ・公開用 |

## よく編集するファイル

| ファイル | 用途 |
|---|---|
| `data/shops.json` | 全店舗情報・募集ポジション（最頻出編集） |
| `index.html` | TOPページ |
| `graduate.html` | 新卒採用ページ |
| `interview-01-03.html` | スタッフインタビュー |
| `shops/*.html` | 店舗詳細ページ（自動生成、直接編集しない） |
| `gas/Code.gs` | 応募フォームのバックエンド（Google Apps Script） |
| `js/application-form.js` | フォームのクライアント側JS |

## ビルドスクリプト

| コマンド | 効果 |
|---|---|
| `node scripts/build-shops.js` | shops.json から店舗ページHTML自動生成 |
| `node scripts/inject-shop-options.js` | フォーム内の店舗セレクトを更新 |
| `node scripts/build-sitemap.js` | sitemap.xml を再生成 |
| `npm run build` | 上記すべて連続実行 |

## 利用可能なスキル

- **recruit-shop-edit**: 店舗情報の編集・追加・削除・募集停止/再開
  - 「時給を変えて」「店舗追加」「募集停止」等の依頼で自動発動
  - 詳細: `.claude/skills/recruit-shop-edit/SKILL.md`

## 応募フォームのバックエンド

- **方式**: Google Apps Script (GAS) Webhook
- **Apps Script プロジェクト名**: SMSA応募フォームWebhook
- **Web App URL**: `js/config.js` の `FORM_ENDPOINT` に記述済み
- **スプレッドシート**: SMSA採用応募データ（ID: `1YbheSsOIAxqkIV6xGhTp9_mPhgQ3D-XorQcR380zMT4`）
- **通知先メール**: スプレッドシートの「設定」シート A列で管理（コード修正不要で追加可能）
- **応募者宛 FROM**: `corporate.division@sports-mario.jp`

## WEB会社説明会 機能（2026-06実装）

- 新卒応募時に「説明会希望日（毎週水曜15:00〜）」を選択可能
- 応募完了時に GAS が:
  1. Google Calendar の該当週イベントに応募者をゲスト自動追加
  2. 応募者にMeet URLを含む案内メール送信
  3. 採用担当に「現在 N 名」の参加人数を通知
- **Google Meet URL**: `https://meet.google.com/uiz-cdhs-vnw`（毎週同じURLで運用）
- **Calendar**: yusuke.kirihara@sports-mario.jp の個人カレンダー
- **除外日設定**: `js/application-form.js` の `EXCLUDED_SEMINAR_DATES` 配列で管理

## 旧採用サイトからのリダイレクト

- 旧URL `sportsmario.co.jp/recruit/*` は XServer の `.htaccess` で 301リダイレクト設定済み
- 設定: `docs/htaccess-redirect.txt`（参考用）

## 重要な禁止事項

1. **`shops/*.html` を直接編集しない** — 自動生成されるファイルです。`data/shops.json` を編集してビルドしてください
2. **`id` フィールドを変更しない** — URLが変わり、リンク切れになります
3. **機密情報を git にコミットしない** — リポジトリは public です
4. **`git push origin main` だけで終わらない** — 必ず `git push codecommit main` も実行

## オンボーディング・運用ドキュメント

新メンバー・運用時の参考:

- `docs/ONBOARDING.md` — 新メンバー初期セットアップガイド
- `docs/COMMON_TASKS.md` — よく使う作業の Claude プロンプト例
- `docs/aws-iam-setup.md` — 管理者用：新メンバーAWS権限付与手順
- `docs/redirect-deployment-guide.md` — 旧サイトリダイレクト設置手順
- `README.md` — プロジェクト概要

## 担当・連絡先

- **プロジェクトオーナー**: 桐原 裕輔（yusuke.kirihara@sports-mario.jp）
- **AWSオーナーアカウント**: 桐原さんで管理
- **応募窓口メール**: corporate.division@sports-mario.jp

---
> Source: [sportsmario-jp/recruit-site](https://github.com/sportsmario-jp/recruit-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
