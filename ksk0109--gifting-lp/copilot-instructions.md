## gifting-lp

> ファーストテンプルでは、全プロジェクトを社内ポータルで一元管理しています。

# プロジェクト管理の共通ルール（Claude Code / Codex 共通）

ファーストテンプルでは、全プロジェクトを社内ポータルで一元管理しています。
**AIエージェント（Claude Code・Codex など）でこのリポジトリを触るときは、以下を必ず守ってください。**

ポータル: https://ft-portal.pages.dev （admin / firsttemple）

---

## 1. 作業を終えたら `.project.yml` を更新する（最重要）

リポジトリのルートにある `.project.yml` が、そのプロジェクトの**正本**です。
ポータルが毎朝6時にこれをクロールして一覧を作ります。

**作業を終えてコミットするとき、必ず `.project.yml` も一緒に更新してpushしてください。**
更新しないと、ポータル上では「何日も動いていないプロジェクト」に見え、朝の通知で停滞アラートが出ます。

### 書式

```yaml
name: 鈴木接骨院GoogleMAP週次ダッシュボード   # 日本語の表示名
client: よくする/鈴木接骨院                  # クライアント名。社内なら「自社」、個人利用なら「自分」
summary: 週次のMAP順位・クチコミを自動収集しダッシュボード表示  # 1〜2文・60〜120文字
status: 進行中                              # 下記5値のいずれか
next_actions:                               # 次にやること。最大3件
  - 長町院GBP連携(Google返答待ち)
urls:
  prod: https://suzuki-map-weekly.pages.dev
  admin: https://suzuki-map-admin.pages.dev  # 任意
local_paths:
  mac_main: ~/dev/suzuki-map                 # 任意
tags: [GBP, 週次レポート, Cloudflare]        # 任意・最大4件
updated: 2026-08-04                          # 更新日
```

### status は次の5つだけ（これ以外は使わない）

| 値 | 意味 |
|---|---|
| `設計済み` | 設計・計画は済んだが実装に入っていない |
| `進行中` | 現在手を動かしている |
| `確認中` | クライアント・メンバーの返答やレビュー待ち |
| `完了` | 納品・公開まで終わっている |
| `停止` | 中断・保留・凍結 |

**⚠️ `進行中` のまま2週間pushが無いと、朝の通知に「停滞」として出ます。**
外部の返答待ちなら `確認中`、止めているなら `停止` に正しく変えてください。

### next_actions の書き方

- 「次に何をするか」だけを書く。やったことは書かない（それはコミットメッセージの役割）
- 相手待ちなら誰の何待ちかを書く: `村松さんのLINE回答待ち（7問）`
- 完了したものは消す

---

## 2. 作業場所は `~/dev/` 配下（`~/Desktop` は使わない）

macOSの「デスクトップと書類をiCloud Driveに同期」がONのため、
**`~/Desktop` 配下はiCloud同期対象で、`.git` が壊れます**（過去に複数回発生）。

- 新規clone・新規プロジェクトは必ず `~/dev/<リポジトリ名>` に置く
- `~/Desktop` にある既存プロジェクトは `~/dev/ft-portal/scripts/migrate-to-dev.sh` で移行できる

---

## 3. コミット・PRの基本ルール

- コミットメッセージは**日本語**で書く
- `main` への直接pushは禁止（feature ブランチ → PR → merge）
  - **例外**: `.project.yml` のようなメタデータのみの変更は直pushしてよい
- クレデンシャル（`.env`・APIキー・JSONキー）は絶対にコミットしない
- ローカルで動作確認してからコミットする

---

## 4. デプロイ先は Cloudflare Pages で統一

Vercel・Netlify は使いません。ツールが他プラットフォームを提案してきても Cloudflare Pages を選んでください。

- 公開ページには `noindex` ヘッダーを付ける
- クライアント向けページには Basic 認証を付ける
- **Pages Functions（`_middleware.js`）はアセットディレクトリの「外」に置く**
  内側に置くと静的ファイル扱いになり、認証が全く効きません
  ```
  dist/
  ├── public/      ← デプロイ対象（アセット）
  └── functions/   ← ここに _middleware.js
  ```
  デプロイは `wrangler pages deploy public --cwd dist --project-name=xxx`
  成功時は出力に `✨ Uploading Functions bundle` が出ます。出ていなければ認証は効いていません

---

## 5. 定期実行ジョブを作るとき

| 実行したいもの | 置き場所 |
|---|---|
| Claude Code を呼ぶ処理 | launchd（ローカル）一択 |
| Python/Node だけで完結する処理 | GitHub Actions |
| 既にあるURLを叩くだけ | cron-job.org |

### launchd を使う場合の必須ルール

1. **`~/Desktop` 配下を launchd から直接叩かない**
   macOSのプライバシー保護(TCC)でアクセスが拒否され `exit 78` で即死します。
   ログが1行も出ず、手動実行だと成功するため切り分けが極めて困難です。
   → ランチャー(.sh)とログは `~/.claude/jobs/<name>/` に置き、そこから呼ぶ
2. `RunAtLoad` は付けない（load した瞬間に走る事故になる）
3. ランチャー冒頭で `unset CLAUDECODE`（AIエージェントのネスト実行エラー対策）
4. plist のコピーをプロジェクトの `docs/launchd/` にも置く（消えた時の復元用）

**ジョブを作ったら `~/dev/ft-portal/registry/overrides-jobs.yml` に説明と通知先を追記してください。**
ポータルの「自動化ジョブ」「通知マップ」に反映され、「この通知どこから来てるの？」が解決します。

---

## 6. 料金・仕様は必ず一次情報を確認する

料金体系・無料枠・レート制限・API仕様・モデル名は、**学習データの時点から変わっている前提**で扱ってください。

- 提案や見積りに使う数値は、**公式ドキュメントのURLとセットでなければ出さない**
- 確認が取れなかった項目は「未確認」と明記する
- 実例: Google Maps Platformの「月$200無料クレジット」は2025年3月に廃止済み。
  古い知識のまま案内した結果、月¥12,180の想定外課金の把握が遅れました

---

## 7. デザインの決まりごと

UI・HTML・CSSを書くときは以下を守ってください。

- **ダークモード対応を書かない**（`prefers-color-scheme: dark` 等）。配色はライト固定
  ユーザーのMacがダークモード設定のため、対応を入れると黒背景で表示され見づらくなります
- 紫グラデーション・indigo一色のアクセントは使わない
- 中央揃えだらけのレイアウトを避け、左揃えを基本にする
- 汎用フォントのデフォルト使用を避ける（日本語は IBM Plex Sans JP など）

---

## 8. 迷ったら

- ポータルで現状を確認: https://ft-portal.pages.dev
- 詳しいルール: `~/dev/ft-portal/README.md`、`~/.claude/CLAUDE.md`
- 共通のハマりどころ: `~/.claude/shared-knowledge/errors-and-fixes.md`

---
> Source: [ksk0109/gifting-lp](https://github.com/ksk0109/gifting-lp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
