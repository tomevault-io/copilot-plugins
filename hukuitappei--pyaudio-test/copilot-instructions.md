## pyaudio-test

> プロジェクト全体のコーディング規約とルール参照


# 全体コーディングルール

ファイル名: coding-rules.mdc
最終更新日時: 2024年12月現在

## 基本方針
プロジェクトはWindowsで作成・実行するとします。
特に指定がない限り、64bitで動作するものとします。

## 使用技術スタック
- **バックエンド**: Python（LTS最新版）
- **フロントエンド**: Next.js + TypeScript
- **データベース**: 必要に応じて設定
- **スタイリング**: Tailwind CSS

## 言語別詳細ルール
各言語・フレームワークの詳細なルールは以下のファイルを参照してください：

### フロントエンド関連
- **Next.js**: `@nextjs-rules.mdc` - Next.js開発のベストプラクティス
- **TypeScript**: `@typescript-rules.mdc` - TypeScript開発規約
- **Python**: `@python-rules.mdc` - Python開発規約（既存）

### 品質管理
- **エラー予防**: `@error-prevention.mdc` - エラー予防とコード品質向上ルール
- **Streamlit デプロイ**: `@streamlit-deployment.mdc` - Streamlit Cloud デプロイメントルール

## 全体共通ルール

### セキュリティ
- 環境変数を使用して機密情報を管理
- APIキーやパスワードをハードコーディング禁止
- 入力値の検証とサニタイゼーション必須
- HTTPS通信を基本とする

### コード品質
- ESLint・Prettierの設定必須
- 型安全性を最優先
- 単体テストカバレッジ80%以上を目標
- コードレビュー必須

### パフォーマンス
- バンドルサイズの監視
- 画像最適化の実装
- 遅延読み込み（Lazy Loading）の活用
- 適切なキャッシュ戦略

### アクセシビリティ
- WCAG 2.1 AA準拠
- セマンティックHTMLの使用
- キーボード操作対応
- スクリーンリーダー対応

### エラーハンドリング
- 適切な例外処理の実装
- ユーザーフレンドリーなエラーメッセージ
- エラーログの標準化
- 障害時の適切なフォールバック

### ドキュメント
- README.mdの詳細記述
- API仕様書の作成
- コメントは日本語で記述
- 変更履歴の適切な管理

### 開発環境
- Git hooks の適切な設定
- 環境別設定ファイルの分離
- ホットリロード機能の活用
- Docker使用時はWindows環境最適化

## ファイル・ディレクトリ命名規則
- **コンポーネント**: PascalCase.tsx
- **ユーティリティ**: camelCase.ts
- **定数**: UPPER_SNAKE_CASE
- **ディレクトリ**: kebab-case
- **設定ファイル**: 各フレームワークの規約に従う

## 追加ルールの作成指針
新しい言語やフレームワークを導入する場合は、対応するルールファイルを作成し、
このファイルから参照する形式を維持してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hukuitappei) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
