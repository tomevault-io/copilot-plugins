## cit

> このファイルは Claude Code がリポジトリの文脈を理解するための設定ファイルです。

# Claude Code 設定

このファイルは Claude Code がリポジトリの文脈を理解するための設定ファイルです。

## プロジェクト概要

GitHub運用とClaude Code連携のためのツール集です。個人開発での効率化を第一目標としています。

## 開発環境

- 言語: 未定（プロジェクトに応じて動的に対応）
- CI/CD: GitHub Actions
- 依存関係管理: Dependabot

## 重要なファイル・ディレクトリ

- `.github/workflows/`: GitHub Actionsワークフロー
  - `code-quality.yml`: 多言語対応のコード品質チェック
  - `auto-assign.yml`: PR自動アサイン
  - `auto-label.yml`: 自動ラベル付け
  - `dependabot-auto-merge.yml`: Dependabot PR自動マージ
- `.github/dependabot.yml`: 依存関係更新設定
- `.github/ISSUE_TEMPLATE/`: イシューテンプレート
  - `bug_report.md`: バグ報告
  - `feature_request.md`: 機能要求
  - `claude_task.md`: Claude Code指示（推奨）
- `.github/pull_request_template.md`: PRテンプレート

## コマンド

このリポジトリはマルチ言語対応のため、具体的なコマンドは検出された言語に応じて動的に決定されます。

## 開発ガイドライン

- **カジュアルな運用**: 完璧性より実用性を重視
- **自動化優先**: 手動作業を最小限に
- **Claude Code活用**: イシューやPRから直接指示を出す

## Claude Code使用例

### 基本的な指示
```
@claude テストを追加してください
@claude コードをリファクタリングしてください
@claude ドキュメントを更新してください
```

### より具体的な指示
```
@claude エラーハンドリングを改善し、適切なログ出力を追加してください
@claude パフォーマンステストを追加し、ボトルネックを特定してください
@claude セキュリティ脆弱性をチェックし、必要な修正を実施してください
```

## 注意事項

- Claude Code GitHub App がインストール済み
- GitHub Actionsの権限設定を適切に行うこと
- 自動マージ機能は慎重に使用すること

## 開発・運用ルール

### コードベース整理の原則
1. **適切な変更単位**: 機能追加、バグ修正、リファクタリングなど、論理的にまとまった単位で変更を行う
2. **即座の文書化**: 変更後は必ずREADME.mdとCLAUDE.mdを最新状態に更新する
3. **構造の同期**: 実際のファイル構成とドキュメントの記述を一致させる
4. **変更の追跡**: 削除したファイルや変更した設定は文書から適切に削除・更新する

### ドキュメント更新のタイミング
- ファイルの追加・削除時
- 設定の変更時
- 機能の追加・削除時
- 運用フローの変更時

### ドキュメント構成の原則
1. **高レベル文書の簡潔性**: README.mdは概要とリンクに留める
2. **詳細の分離**: 具体的な設定や手順は専用ディレクトリのREADME.mdに記載
3. **適切な分割**: 
   - `.github/workflows/README.md` - GitHub Actionsの詳細
   - `.github/ISSUE_TEMPLATE/README.md` - テンプレートの詳細
   - その他必要に応じて分割
4. **冗長性の排除**: 同じ情報の重複記載を避け、リンクで参照
5. **読みやすさ優先**: 長すぎる説明は分割し、要点を明確に

このルールにより、コードベースと文書が常に同期し、可読性と保守性を両立する。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hikuohiku) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
