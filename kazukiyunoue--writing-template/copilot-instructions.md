## writing-template

> このリポジトリは、AI（Claude Code）による執筆支援のためのテンプレートです。

# 執筆支援プロジェクト

このリポジトリは、AI（Claude Code）による執筆支援のためのテンプレートです。

## ディレクトリ構成

```
ideas/          # アイディアメモ（日時付きマークダウン）
research/       # 調査・リサーチ結果
drafts/         # 原稿（草稿）
project.md      # プロジェクト情報（タイトル、概要など）
outline.md      # 作品のアウトライン
```

## ワークフロー

起点は `/research` でも `/idea` でも構わない。リサーチで集めた情報を `/idea` でインサイト化し、両者を往復しながら材料を整えていく。

1. **調査・リサーチ** (`/research`) - 必要な情報をバックグラウンドで収集し、`research/` に保存
2. **アイディアの発散** (`/idea`) - 思いついたことやリサーチから得たインサイトを `ideas/` に記録
3. **アイディアの反映** (`/reflect`) - アイディアを整理し、outline.md や research/ へ反映
4. **原稿の執筆** (`/draft`) - outline.md をもとに drafts/ で執筆（手動執筆も可。`/draft` を一切使わない運用も想定）
5. **校正** (`/proofread`) - 誤字脱字・文法のチェック
6. **校閲** (`/review`) - 内容の整合性・事実確認

### /feedback によるブラッシュアップ

`/feedback` は対象ファイルの各行を引用形式に変換したアイディアファイルを作成する。引用にコメントを追記して `/reflect` を実行することで、コメントが該当ファイルへ反映される。

このループはアウトライン段階・草案段階のどちらでも回せる：

- アウトライン段階: `/feedback outline.md` → コメント追記 → `/reflect`
- 草案段階: `/draft` → `/feedback drafts/01.md` → コメント追記 → `/reflect`

## フィクション執筆の場合

人物設定や舞台設定が必要な場合は、`settings/` ディレクトリを作成して管理できます。

```
settings/
  characters/   # 人物設定
  world/        # 舞台・世界観設定
```

## AIへの指示

- ユーザーの文体・トーンを尊重すること
- 勝手に内容を大幅に変更しないこと
- 提案は明確に提案として示し、承認を待つこと
- 校正・校閲では、指摘箇所と理由を明示すること

---
> Source: [KazukiYunoue/writing-template](https://github.com/KazukiYunoue/writing-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
