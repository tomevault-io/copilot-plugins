## consultingtool

> コンサルティングファームが複数のクライアントプロジェクトを効率的に管理・監視するための統合ダッシュボードシステム。プロジェクトの進捗状況、リソース配分、財務状況を一元的に可視化し、意思決定を支援する。

# コンサルティングプロジェクトダッシュボード

## アプリケーション概要

コンサルティングファームが複数のクライアントプロジェクトを効率的に管理・監視するための統合ダッシュボードシステム。プロジェクトの進捗状況、リソース配分、財務状況を一元的に可視化し、意思決定を支援する。

## アーキテクチャスタック

- **フロントエンド**: Next.js 15.1.0 (App Router), React 19.0, TypeScript 5.7, Tailwind CSS 3.4.1
- **UI コンポーネント**: shadcn/ui (MCP経由)
- **バックエンド**: Next.js Server Actions
- **データベース**: SQLite (ファイルベースDB)
- **ORM**: Prisma 6.0
- **バリデーション**: Zod (スキーマベースバリデーション)
- **フォーム管理**: React Hook Form (Zod連携)

## ドキュメント構成

このプロジェクトのドキュメントは、メンテナンス性と可読性を考慮して以下のように分割されています：

### アーキテクチャ関連
- [プロジェクトディレクトリ構造](docs/architecture/directory-structure.md)
- [データベース構成](docs/architecture/database-configuration.md)
- [環境変数の設定](docs/architecture/environment-setup.md)

### 開発関連
- [Git開発フローとissue管理](docs/development/git-workflow.md)
- [コミットメッセージ規約](docs/development/commit-conventions.md)
- [動作確認手順](docs/development/testing-guidelines.md)

### UI/UX関連
- [ナビゲーションシステム仕様](docs/ui-ux/navigation-system.md)
- [認証とロール管理](docs/ui-ux/authentication.md)
- [主要ユースケース](docs/ui-ux/use-cases.md)

### パラソル開発手法
- [パラソル開発手法の概要](docs/parasol/methodology.md)
- [ビジネスオペレーションとユースケース](docs/parasol/business-operations.md)
- [命名規則](docs/parasol/naming-conventions.md)
- [設計MD作成ガイド](docs/parasol/design-md-guide.md)
- [API経由での設計MDポスト](docs/parasol/api-integration.md)

## 設計書について

設計書は`design`ディレクトリに配置され、複数あるものはディレクトリを作成して管理します。各設計書には更新日を記載してください。

- プロジェクト定義
- ビジネスケーパビリティ定義
- ユースケース（ビジネスケーパビリティをブレークダウンしてユースケースを生成）
- ロバストネス図
- API定義
- UI定義
- ドメイン言語モデル（MD形式）
- DB定義
- 受け入れテスト
- 単体テスト
- イテレーション計画

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hmoriya)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hmoriya)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
