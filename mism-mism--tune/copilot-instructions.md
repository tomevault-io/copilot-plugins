## git-commit-branch

> コミットメッセージ形式とブランチ名の慣習。PR は gh で作成する。


# Git コミット・ブランチルール

## コミット

- **1 変更 1 コミット** — 論理的にまとまる単位でコミットする
- **メッセージ形式**: `種別(スコープ): 説明`
  - 種別: feat / fix / docs / refactor / test / chore
  - スコープ: domain, application, infrastructure, ui 等（任意）
  - 説明は日本語可

例: `feat(domain): User と UserId を追加`, `fix(auth): セッション期限切れ時のリダイレクト`

## ブランチ

- **命名**: `feature/短い識別子` または `fix/短い識別子`
- 例: `feature/task-1-2-user-entity`, `fix/login-session`
- 1 機能 or 1 タスク（development-tasks の 1 項目）ごとに 1 ブランチを目安にする

## PR の作成・確認

- **GitHub CLI (`gh`) を使う**
- **マージ先は既定で `main`**: `gh pr create --base main`（`--fill` / `--title` / `--body` と併用）
- 一覧: `gh pr list`
- 確認: `gh pr view`
- ベース付け替え: `gh pr edit <番号> --base main`（コンフリクト時はブランチに `origin/main` を merge）

詳細は `git-workflow` と `pr-creation` スキルを参照。

---
> Source: [mism-mism/tune](https://github.com/mism-mism/tune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
