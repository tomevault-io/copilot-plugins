## iot-temperature-conversion-api

> IoT業界における温度データの単位変換問題を解決し、異なるセンサー・システム間のデータ互換性を提供する専門特化APIサービス。

# IoT Temperature Conversion API

## プロジェクト概要

IoT業界における温度データの単位変換問題を解決し、異なるセンサー・システム間のデータ互換性を提供する専門特化APIサービス。

**プロジェクト名**: TempConvert API
**目標**: 12ヶ月以内に月間収益$8,000達成、IoT開発企業800社の顧客基盤構築

## 知見管理システム
このプロジェクトでは以下のファイルで知見を体系的に管理しています：

### `.claude/context.md`
- IoT温度変換API の背景、目的、制約条件
- Node.js + TypeScript + PostgreSQL技術スタック選定理由
- ビジネス要件（99.9%アップタイム、1ms以下レスポンス）や技術的制約

### `.claude/project-knowledge.md`
- 温度変換ロジックの実装パターンや設計決定の知見
- RESTful API設計とマイクロサービスアーキテクチャの選択理由
- セキュリティ実装（JWT + API Key）とレート制限のパターン
- 避けるべきパターンやアンチパターン（精度の丸め誤差、同期処理など）

### `.claude/project-improvements.md`
- バッチ処理最適化の試行錯誤記録
- Vercel → AWS移行時の失敗実装とその原因
- スケーリング対応の改善プロセスと結果
- API精度要件（NIST標準準拠）の実装改善履歴

### `.claude/common-patterns.md`
- 温度変換APIの頻繁に使用するコマンドパターン
- 認証ミドルウェアの定型的な実装テンプレート
- エラーハンドリング（RFC 7807準拠）のパターン
- テストコード（90%カバレッジ）の標準パターン

### `ARD/`（20250629追加）
- アーキテクチャ記録文書（Architecture Record Documents）
- 重要な技術決定や実装の背景を記録
- ファイル命名規則：`ARD-YYYYMMDD-brief-description.md`
- 例：`ARD-20250629-temperature-precision-strategy.md`
- 新機能設計・既存機能変更・技術課題解決時に作成

**重要**: 新しい実装や重要な決定を行った際は、該当するファイルを更新してください。また、変更したタイミングがわかるような工夫をしてください。YYYYMMDDを表記するなど。

ユーザーから今回限りではなく常に対応が必要だと思われる指示を受けた場合：

1. 「これを標準のルールにしますか？」と質問する
2. YESの回答を得た場合、CLAUDE.mdに追加ルールとして記載する
3. 以降は標準ルールとして常に適用する

このプロセスにより、プロジェクトのルールを継続的に改善していきます。

## Git開発フロー標準ルール（20250629追加）

機能開発時は以下のブランチ管理フローを必ず適用する：

1. **機能開発開始時**
   - `git checkout -b feature/機能名` でブランチを作成
   - ブランチ名例：`feature/batch-conversion`, `feature/jwt-auth`, `feature/rate-limiting`
   - ブランチ名は機能内容が分かりやすい名前にする

2. **開発中**
   - 適切な粒度で定期的にコミット
   - コミットメッセージは変更内容が明確に分かるよう記載
   - 例：`feat: implement Celsius to Fahrenheit conversion logic`
   - 例：`fix: resolve precision rounding error in Kelvin conversion`

3. **機能完成時**
   - `git push origin feature/機能名` でリモートにブランチをプッシュ
   - プルリクエスト作成時はAPIテスト結果とパフォーマンステスト結果を添付
   - 必要に応じてプルリクエスト作成やmainブランチへのマージを検討

4. **開発完了後**
   - ローカルのfeatureブランチは削除
   - mainブランチに戻って次の開発に備える

## ファイル配置標準ルール（20250629追加）

プロジェクトディレクトリの整理と保守性を確保するため、以下のファイル配置規則を厳守する：

### 1. **テストファイル**
- **配置先**: `tests/` ディレクトリ
- **命名規則**: `test-{機能名}.ts`
- **例**: `tests/test-temperature-conversion.ts`、`tests/test-batch-processing.ts`、`tests/test-api-auth.ts`

### 2. **ドキュメントファイル**
- **配置先**: `docs/` ディレクトリ
- **API仕様**: `docs/api/openapi.yaml`
- **開発ガイド**: `docs/development/setup-guide.md`
- **SDK仕様**: `docs/sdk/javascript-sdk.md`、`docs/sdk/python-sdk.md`

### 3. **設定ファイル**
- **配置先**: `config/` ディレクトリ
- **環境別設定**: `config/development.json`、`config/production.json`、`config/staging.json`
- **データベース設定**: `config/database.config.ts`
- **Redis設定**: `config/redis.config.ts`

### 4. **スクリプトファイル**
- **配置先**: `scripts/` ディレクトリ
- **ビルドスクリプト**: `scripts/build.sh`
- **デプロイスクリプト**: `scripts/deploy-vercel.sh`、`scripts/deploy-aws.sh`
- **マイグレーションスクリプト**: `scripts/migrate-database.ts`

### 5. **SDKファイル**
- **配置先**: `sdk/` ディレクトリ
- **JavaScript SDK**: `sdk/javascript/temp-converter-sdk.js`
- **Python SDK**: `sdk/python/temp_converter_sdk.py`
- **Go SDK**: `sdk/go/temp_converter.go`

### 6. **一時ファイル・実験ファイル**
- **配置先**: `temp/` ディレクトリ（.gitignoreに追加）
- **プロトタイプ**: `temp/prototype-precision-test.ts`
- **負荷テスト**: `temp/load-test-results.json`

**重要**: ルートディレクトリに一時ファイルやテストファイルを直接配置することを禁止する。これはプロジェクトの可読性と保守性を著しく損なうため。

## API開発標準ルール（20250629追加）

温度変換APIの品質と一貫性を確保するため、以下の開発ルールを厳守する：

### 1. **レスポンス時間**
- 単一変換：1ms以下（95%パーセンタイル）
- バッチ変換：10ms以下（1,000データポイント）
- パフォーマンステストは機能実装完了時に必須実行

### 2. **精度要件**
- NIST標準準拠の変換式を使用
- 有効数字15桁の精度を保持
- コンテキスト別精度設定（消費者用/工業用/医療用/科学用）

### 3. **エラーハンドリング**
- RFC 7807準拠のエラーレスポンス形式
- 適切なHTTPステータスコード使用
- 詳細なエラーメッセージとエラーコード提供

### 4. **セキュリティ**
- TLS 1.3以上の暗号化必須
- API Key + JWT認証の実装
- レート制限とIP制限の適用
- OWASP API Top 10対応

**重要**: 新しいエンドポイント追加時は、上記ルールの遵守とOpenAPI仕様書の更新を必須とする。

## 開発フロー標準ルール（20250629追加）

すべての開発作業は以下の標準化されたフローに従って実行する：

### 1. **開発開始前の必須確認事項**
- `docs/development-flow.md` で最新の開発フローを確認
- `.claude/` ディレクトリの知見ファイルを読み込み
- `ARD/` ディレクトリの技術決定記録を確認
- 既存のルールやパターンとの整合性を検証

### 2. **新機能開発時の必須プロセス**
- 重要な技術決定にはARDファイルを作成（`ARD-YYYYMMDD-brief-description.md`）
- 品質ゲートの遵守（パフォーマンス要件、テストカバレッジ90%以上）
- 適切なディレクトリ配置（tests/, docs/, config/, scripts/, sdk/）

### 3. **コード品質要件**
- NIST標準準拠の温度変換精度（有効数字15桁）
- RFC 7807準拠のエラーハンドリング
- TLS 1.3以上、API Key + JWT認証、レート制限の実装

### 4. **ドキュメント更新義務**
- 新しい実装や重要な決定時は知見ファイル更新
- 更新時は必ず日付（YYYYMMDD）を記録
- 変更履歴の追跡可能性を確保

### 5. **継続的改善プロセス**
- 新しい標準ルール追加時は必ずユーザー確認を取る
- 承認されたルールは本ファイルに追記し、以降標準適用

**重要**: このフローに従わない開発は品質リスクとなるため、必ず遵守すること。詳細は `docs/development-flow.md` および `ARD/ARD-20250629-development-flow-standardization.md` を参照。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Clionegohan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Clionegohan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
