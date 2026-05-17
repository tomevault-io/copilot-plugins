## isohyps

> このリポジトリでは、CLIエージェントに「1工程ずつ」仕事をさせる。

# AGENTS.md

このリポジトリでは、CLIエージェントに「1工程ずつ」仕事をさせる。

目的は、単発の巨大プロンプトで全部やらせるのではなく、
以下の工程を順番に実行し、各工程の成果物を明示的に残すこと。

1. prototype planning
2. prototyping
3. red team review
4. planning
5. implementation
6. review/fix loop
7. pull request

---

## 基本ルール

- 常に **Issue #xx** を起点に作業する。
- Issue の主ソースは **GitHub Issues** を優先する。
- 必要に応じて Issue コメントも入力に含める。
- 1回の工程では **その工程の責務だけ** を実施する。
- 次工程に必要な情報は、必ず成果物として `.kelpie/artifacts/` に残す。
- 不明点があっても作業停止を最小化し、妥当な仮定を明示して前進する。
- 破壊的変更・依存追加・権限変更・外部送信を伴う場合は理由を成果物に記録する。
- 実装より前に、少なくとも一度は失敗条件・非目標・既知リスクを書く。
- 各工程では、対応する `skills/<phase>/SKILL.md` を必ず読む前提で行動する。
- 各工程終了時に、最低限以下を出力する:
  - 何をやったか
  - 何をやっていないか
  - 次工程への入力
  - リスク / 未解決事項

---

## ディレクトリ規約

```text
.kelpie/
  .gitignore
  instructions/
  artifacts/
    github/
      owner/
        repo/
          issue-xx/
            01-prototype-planning.md
            02-prototype-summary.md
            03-red-team-review.md
            04-plan.md
            05-implementation-notes.md
            06-review-fix-loop.md
            07-pr-draft.md
            .issue-cache/
              issue.json
              issue_comments.json
            intent-records/
            checks/
```

`.kelpie/artifacts/.../issue-xx/` 配下に工程ごとの成果物を残す。

---

## 工程ごとの責務

### 1) prototype planning
目的:
- 問題の理解を揃える
- 最小プロトタイプの範囲を決める
- 成否判定を簡単に定義する

出力:
- `01-prototype-planning.md`

やること:
- GitHub Issue を要約
- 要件 / 非要件 / 仮定 / リスク整理
- 最小スパイク案を 1〜3 個提案
- 最初に作る試作品を 1 つに絞る
- 何を捨てるか明記

### 2) prototyping
目的:
- 捨てやすい実験で見通しを得る

出力:
- `02-prototype-summary.md`
- 必要なら試作コード

やること:
- 本実装前提にせず、最短で仮説検証
- 設計の美しさより学習速度を優先
- 試したこと / わかったこと / 無理だったことを記録

### 3) red team review
目的:
- 試作や計画の危険点を先に炙る

出力:
- `03-red-team-review.md`

やること:
- 想定破綻点、誤用、過信、見落としを列挙
- 仕様の穴、セキュリティ、運用事故、UX事故、保守性を点検
- 「このまま進めるなら最低限必要なガード」を提案

### 4) planning
目的:
- 実装可能な計画に落とす

出力:
- `04-plan.md`

やること:
- タスク分解
- 依存関係整理
- 実装順序決定
- 完了条件と確認方法の明記

### 5) implementation
目的:
- 計画に従って実装する

出力:
- コード変更
- `05-implementation-notes.md`

やること:
- 1回で全部盛りしない
- 計画との差分が出たら理由を書く
- 追加したファイル、主要変更点、未対応点を残す

### 6) review/fix loop
目的:
- 実装品質を上げる

出力:
- `06-review-fix-loop.md`

やること:
- レビュー
- 問題の優先度付け
- 修正
- 再確認
- ループを定義回数または収束条件まで回す

### 7) pull request
目的:
- 人間がレビューしやすいPR材料を揃える

出力:
- `07-pr-draft.md`

やること:
- 変更概要
- 背景
- 変更点
- テスト
- 残課題
- レビューポイント

---

## エージェントへの共通指示

- 大きく迷ったら、抽象議論を伸ばしすぎず、現時点で妥当な選択肢を1つ採る。
- 不確実性は消さずに記録する。
- できるだけ小さい差分を積む。
- GitHub Issue 本文とコメントのうち、実装判断に効く文脈を優先して使う。
- Issue だけで足りない場合は、リポジトリ内の関連コード・既存ドキュメント・過去成果物で補完する。
- 自動チェック可能な点は常に意識する。
- 将来 `Intent Record` と `機械チェック` が差し込まれる前提で、判断理由を工程単位で残す。

---

## 各工程で参照する入力

- GitHub Issue 本文
- 必要に応じて GitHub Issue コメント
- `AGENTS.md`
- 対応工程の `skills/<phase>/SKILL.md`
- 対応工程の `prompts/*.md`
- それ以前の工程で生成された `.kelpie/artifacts/.../issue-xx/*`

---

## 失敗時の扱い

- 失敗を隠さない。
- 途中で詰まった場合も、
  - どこまで進んだか
  - 何が障害か
  - 次に人間が判断すべき点
  を成果物に残して終了する。

---
> Source: [aoitan/isohyps](https://github.com/aoitan/isohyps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
