## diary-daily-report

> 日報（diary）の作成ルールとファイル名・フォルダの指定


# 日報（diary）の作成ルール

**日報**を追加するときは、必ず次のルールに従うこと。

## フォルダ

- **日報は `diary/` ディレクトリに格納する。**

## ファイル名

- **形式**: `YYYYMMDD.md`（日付のみ、ハイフンや接頭辞は付けない）
- **例**: `20260201.md`、`20260215.md`

日付は日報を書く日（当日）の YYYYMMDD を用いる。

## 作業日報・完了タスクの記録

- **「作業日報」や完了した TODO の記録も、上記と同じ `diary/YYYYMMDD.md` に追記する。**
- `docs/` などに「作業日報.md」のような単一ファイルを置かず、日付ごとの日報ファイル（`diary/YYYYMMDD.md`）に作業内容・完了タスクを書く。
- TODO を更新するときは、完了項目を当日の `diary/YYYYMMDD.md` に追記し、TODO から当該項目を削除する（詳細は `todo-update-with-diary.mdc` を参照）。

## 作成前の確認

- **日報作成前に、必ず `date` コマンド（または同等の方法）で当日の日付を確認すること。**
- 例: `date +%Y%m%d` で YYYYMMDD を取得し、ファイル名に使う。

このルールは、日報を新規作成するときに必ず適用すること。

---
> Source: [nistake0/cursor01-athletic-game](https://github.com/nistake0/cursor01-athletic-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
