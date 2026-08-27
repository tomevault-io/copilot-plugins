## runmates

> あいまいな点や不明点があれば、AskUserQuestionツールを使って質問してください。

# CLAUDE.md
必ず日本語で回答してください。
あいまいな点や不明点があれば、AskUserQuestionツールを使って質問してください。

## ⚠️ 最重要: Dockerコンテナ内実行の原則

**すべてのコマンドは必ずDockerコンテナ内で実行してください。**

```bash
# ✅ 正しい例（必ずこの形式で実行）
docker compose exec rails bundle exec rubocop
docker compose exec rails rails console
docker compose exec next npm run dev

# ❌ 間違った例（絶対に実行しない）
bundle exec rubocop  # ローカル実行
rails console       # ローカル実行
npm run dev        # ローカル実行
```

**例外（ローカル実行OK）:**
- `gh` コマンド: Runmatesリポジトリ直下から直接実行（コンテナ不要）
- `terraform` コマンド: `terraform/envs/prod` ディレクトリでローカル実行

## ブランチ命名規則

```
main              # 本番環境（保護ブランチ）
├── feature/*     # 新機能開発
├── fix/*         # バグ修正
├── refactor/*    # リファクタリング
```

**重要:** この3種類のみ使用。`chore/*` や `docs/*` は使わない。

## フック（hooks）

Claude Code 実行時に `.claude/hooks/scripts/` 配下のフックが自動発火する（コミット前チェック・破壊コマンド遮断・秘密情報スキャン・ブランチ警告）。
詳細・ヘッダ規約は [`.claude/rules/hooks.md`](.claude/rules/hooks.md) 参照（`.claude/hooks/**` / `.claude/settings.json` 編集時に自動ロード）。

## Playwright MCPによる動作確認

実装完了後、UI/APIに影響する変更がある場合はPlaywright MCPで動作確認を行う。

- テストアカウント: `test@example.com` / `password123`（`rails db:seed`で作成）
- 対象: フォーム送信、画面遷移、エラー表示など、ユーザー操作に関わる変更
- **確認が終わったら必ず `browser_close` でブラウザを閉じること**

### モバイルサイズでの確認

UI/レスポンシブに影響する変更がある場合、デスクトップでの確認に加えてモバイルサイズでも確認する。

1. デスクトップサイズで動作確認を実施
2. `browser_resize` でモバイルサイズ（width: 375, height: 667）に切り替え
3. 以下の観点で確認:
   - レイアウト崩れがないか
   - タップ操作で正常に動作するか
   - ナビゲーション/メニューが正しく表示されるか
   - フォームの入力・送信が問題なくできるか
4. 確認完了後、`browser_close` でブラウザを閉じる

## git add の禁止事項

**`git add` はユーザーの明示的な承認なしに絶対に実行しない。**
修正作業・レビュー対応・コマンド実行のいずれの場合でも、必ず `git status` と `git diff` で変更内容を提示し、ユーザーの承認を得てから `git add` を実行すること。

## 実装後のチェックフロー

1. リント・テストを実行して通ることを確認（`rubocop`、`rspec`、`npm run lint`）
2. `/rev` でセルフレビュー（必要性チェック含む）
3. 「作業完了しました」と報告
4. ユーザーの承認を得てから `git add` を実行

## Issue/PRテンプレート

- **バグレポート**: `.github/ISSUE_TEMPLATE/bug_report.md` - `[Bug]` プレフィックス
- **機能要望**: `.github/ISSUE_TEMPLATE/feature_request.md` - `[Feature]` プレフィックス
- **PR**: `.github/PULL_REQUEST_TEMPLATE.md` - 関連Issueを `Fixes #番号` で紐付け

## GitHub CLI

- `gh`コマンドでコメントを付与するときはマークダウン形式で作成すること
- 認証: 環境変数 `GH_TOKEN`（最優先）、keyring/キーチェーン（推奨）
- トークン期限切れ時: `gh auth logout` → `gh auth login` でブラウザ認証

## レビュールール

- レビューには解決案も付ける
- コードレビューでは、セキュリティ・パフォーマンス・可読性を重視

## トークン使用量削減

- 大きなファイルは`offset`と`limit`パラメータを使用
- コードの動作説明は不要（ユーザーが要求した場合のみ）
- 複数の独立したタスクは一度に並列実行

## ルール構成

詳細なコーディング規約は `.claude/rules/` 参照（パス条件で自動ロード）。

---
> Source: [hata-junnosuke/runmates](https://github.com/hata-junnosuke/runmates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
