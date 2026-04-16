## account-book

> このファイルはCursor IDEがプロンプト実行時に自動的に読み込みます。

# Cursor Rules - Account Book Project

このファイルはCursor IDEがプロンプト実行時に自動的に読み込みます。

---

## ルールファイルの自動参照

以下のルールファイルを**必ず参照**してください：

### 0. `.cursor/rules/00-workflow-checklist.d/` ⭐ ワークフロー・チェックリスト

**最優先ガイド - 全体フローの羅針盤**

- タスク開始から完了までの全体フロー
- 各ステップで参照すべきルールファイル
- 最優先事項のサマリー
- クイックリファレンス

**参照**: `.cursor/rules/00-workflow-checklist.d/README.md`（ディレクトリの説明とファイル一覧）

### 1. `.cursor/rules/01-project.d/` - プロジェクト全体のガイドライン

- プロジェクト概要
- 技術スタック
- アーキテクチャ原則（Onion Architecture）
- コーディング規約
- Issue管理

**参照**: `.cursor/rules/01-project.d/README.md`（ディレクトリの説明とファイル一覧）

### 2. `.cursor/rules/02-code-standards.d/` - コード品質基準とテスト

- 型安全性（最優先）⭐
- テスト作成の必須化
- データアクセスと配列操作
- アーキテクチャとモジュール設計
- テスト実装ガイドライン
- ESLint設定
- React/UIコンポーネント
- 実装フローチェックリスト
- スクリプト・ツール開発
- push前の必須チェック
- シェルスクリプトのベストプラクティス
- Geminiレビューから学んだ観点

**参照**: `.cursor/rules/02-code-standards.d/README.md`（ディレクトリの説明とファイル一覧）

### 3. `.cursor/rules/03-git-workflow.d/` - Git ワークフロー

- 🚨 絶対禁止事項（mainブランチ直接作業、PRなしマージ、PRマージ前Issue Done変更）
- ブランチ管理
- コミットルール（最優先）
- Push前チェック（必須）
- PR作成・レビュー

**参照**: `.cursor/rules/03-git-workflow.d/README.md`（ディレクトリの説明とファイル一覧）

### 4. `.cursor/rules/04-github-integration.d/` - GitHub統合

- GitHub Projects設定
- Issueステータス管理
- Issue完了報告（最優先）
- @start-task統合

**参照**: `.cursor/rules/04-github-integration.d/README.md`（ディレクトリの説明とファイル一覧）

### 5. `.cursor/rules/05-ci-cd.d/` - CI/CD設定ガイドライン

- テスト実行スクリプトの設計原則
- 環境変数の管理
- PlaywrightのwebServer設定
- GitHub Actions設定

**参照**: `.cursor/rules/05-ci-cd.d/README.md`（ディレクトリの説明とファイル一覧）

### テンプレート

- `.cursor/rules/templates/issue-report.md` - Issue報告テンプレート
- `.cursor/rules/templates/pr-description.md` - PR説明テンプレート

---

## 重要: すべてのルールの適用

**これらのルールファイルは、プロンプト実行時に自動的に参照され、適用されなければなりません。**

---

## 🚨 最優先で遵守すること

### 🔴 0. Gitワークフロー遵守（最重要）

- ❌ **mainブランチでの直接作業は絶対禁止**
- ❌ **PRなしでのmainへのマージは絶対禁止**
- ❌ **PRマージ前にIssueをDoneに変更することは絶対禁止**
- ✅ **必ずフィーチャーブランチを作成してから作業開始**
- ✅ **すべての変更はPR経由でmainにマージ**
- ✅ **PRマージ後のみIssueをDoneに変更**

**参照**: `.cursor/rules/03-git-workflow.d/01-branch-management.md`（絶対禁止事項セクション）

### 1. 型安全性

- ✅ **any型の使用禁止**（テストのJestモック作成時のみ例外）
- ✅ **Enum型の比較は型安全に**（`Object.entries()`使用時は明示的型キャスト）
- ✅ **危険な型キャストの禁止**
- ✅ **DTO設計**: リクエストは`class`、レスポンスは`interface`（Issue #22で学習）

**参照**: `.cursor/rules/02-code-standards.d/01-01-type-safety.md`

### 2. 自動コミット

- ✅ **作業完了時は即座にコミット**
- ✅ **質問・確認なしで自動実行**
- ❌ **「後でコミット」は禁止**

**参照**: `.cursor/rules/03-git-workflow.d/02-commit-rules.md`

### 3. push前チェック（4ステップ）

- ✅ **必ずlint/build/test/e2eを実行**（⭐ buildが追加されました）

```bash
./scripts/test/lint.sh
pnpm build  # ⭐ ビルドチェックを忘れない！
./scripts/test/test.sh all
./scripts/test/test-e2e.sh frontend
```

**テスト実行の判断基準:**

- **ソースコード変更時**: ステップ1〜4すべて実行（必須）
- **マークダウン/ドキュメントのみ**: ステップ1・2のみでOK（テスト・E2E不要）

詳細は「テスト実行の判断基準」を参照（`.cursor/rules/03-git-workflow.d/03-push-check.md`）

**理由**: 
- pushするとGitHub ActionsでCIが実行される
- **特にビルドエラーはすべてのCI jobをブロックするため、最優先で確認**
- ローカルで事前にエラーを検出することで、無駄なCI実行を防止できる
- 実例: Issue #22でビルドチェック忘れ → CI全滅 → 20分の時間損失

**参照**: `.cursor/rules/03-git-workflow.d/03-push-check.md` / `.cursor/rules/02-code-standards.d/01-03-push-check.md`

### 4. @start-task

- ✅ **最初に必ずすべてのルールファイルを再読込**（@inc-all-rulesの内容を実行）
- ✅ **質問せず即座に最優先タスクを開始**
- ✅ **以下の条件を満たすIssueを選択:**
  - GitHub Projectsで「📝 To Do」ステータス
  - 自分がアサインされている
  - OPENステータス
- ✅ **優先度順にソート（priority: critical > high > medium > low）**
- ✅ **優先度が同一の場合はIssue番号が小さい方を優先**
- ✅ **mainブランチからフィーチャーブランチを作成**
- ✅ **ステータスを「🚧 In Progress」に変更**

**🎯 新機能: start-task.sh スクリプト**

Issue開始を自動化するスクリプトが利用可能です：

```bash
# 最優先Issueを自動選択
./scripts/github/workflow/start-task.sh

# Issue番号を指定して開始
./scripts/github/workflow/start-task.sh #201
```

**参照**: `.cursor/rules/04-github-integration.d/04-start-task.md`

### 5. Geminiレビュー対応

- ✅ **すべての指摘に個別commit/pushで対応**
- ❌ **複数の指摘をまとめて1つのcommitにすることは絶対に禁止**
- ✅ **すべての指摘対応完了後、必ずPRコメントで返信**

**参照**: `.cursor/rules/03-git-workflow.d/04-pr-review.md`

### 6. Issue報告

- ✅ **commit完了後は必ずGitHub Issueに報告**
- ✅ **テンプレートに従って詳細を記載**

**参照**: `.cursor/rules/04-github-integration.d/03-issue-reporting.md`

### 7. Issue作成方法

- ✅ **必ず専用スクリプトを使用**: `./scripts/github/issues/create-issue.sh`
- ❌ **`gh issue create`の直接使用は禁止**（プロジェクトに自動追加されないため）

**参照**: `.cursor/rules/04-github-integration.d/README.md`

---

## 📋 ワークフロー全体像

```
@start-task
    ↓
1️⃣ タスク開始
    ↓
2️⃣ 実装・コミット
    ↓
3️⃣ push前チェック
    ↓
4️⃣ PR作成・レビュー
    ↓
5️⃣ マージ・完了報告
```

**詳細**: `.cursor/rules/00-workflow-checklist.d/README.md`

---

## ⚡ クイックリファレンス

### よく使うコマンド

```bash
# タスク開始
@start-task

# Lintチェック
./scripts/test/lint.sh

# テスト実行
./scripts/test/test.sh all

# E2Eテスト
./scripts/test/test-e2e.sh frontend

# Issue作成（必ずスクリプトを使用）
./scripts/github/issues/create-issue.sh <data-file>

# Issue報告
gh issue comment <ISSUE_NUMBER> --body "<報告内容>"

# PR作成・コメント
gh pr create --title "<タイトル>" --body "<内容>"
gh pr comment <PR_NUMBER> --body "<コメント>"

# ステータス更新
./scripts/github/projects/set-issue-in-progress.sh <ISSUE_NUMBER>
./scripts/github/projects/set-issue-done.sh <ISSUE_NUMBER>
```

---

**注意**: このファイルは `.cursor/rules/` 内の各ルールファイルへの参照を提供します。
詳細なルール内容は各ファイルを確認してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kencom2400) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
