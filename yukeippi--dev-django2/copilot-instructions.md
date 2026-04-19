## dev-django2

> このファイルは、Claude Code（claude.ai/code）がこのリポジトリでコードを扱う際のガイダンスを提供します。

# CLAUDE.md

このファイルは、Claude Code（claude.ai/code）がこのリポジトリでコードを扱う際のガイダンスを提供します。

## 開発コマンド

### 環境設定
```bash
# Poetryを使用して依存関係をインストール
poetry install

# 仮想環境をアクティベート
poetry shell

# 別の方法: poetryプレフィックスでコマンドを実行
poetry run python manage.py [command]
```

### Django開発
```bash
# 開発サーバーを起動
poetry run python manage.py runserver 0.0.0.0:8000

# データベースマイグレーション
poetry run python manage.py makemigrations
poetry run python manage.py migrate

# スーパーユーザーを作成
poetry run python manage.py createsuperuser

# 静的ファイルを収集
poetry run python manage.py collectstatic

# テストを実行
poetry run python manage.py test

# 新しいアプリを作成
poetry run python manage.py startapp <app_name>
```

### テスト
```bash
# 全てのテストを実行
poetry run python manage.py test

# 特定のテストモジュールを実行
poetry run python manage.py test diary
poetry run python manage.py test diary.tests.TestPost
```

## アーキテクチャ概要

### プロジェクト構造
- **config/**: Djangoプロジェクト設定
  - `settings.py`: PostgreSQL、デバッグツールバーを含む開発設定
  - `settings_production.py`: セキュリティ設定を含む本番設定
  - `urls.py`: ルートURL設定
- **apps/**: Djangoアプリケーションのコンテナ
- **diary/**: ブログエントリ用Postモデルを持つメインアプリケーション
- **static/**: 静的ファイル（CSS、JS、画像）
- **deploy/**: EC2用デプロイスクリプトと設定

### 主要設定詳細
- **データベース**: 環境ベース設定のPostgreSQL
- **静的ファイル**: 本番環境での静的ファイル配信にWhiteNoiseを使用
- **ローカライゼーション**: 日本語（`ja`）、Asia/Tokyoタイムゾーン
- **セキュリティ**: 本番設定にはHSTS、XSS保護、コンテンツタイプスニッフィング保護を含む

### 開発環境
- **Python**: 3.11+
- **Django**: 4.2.x
- **データベース**: PostgreSQL（devcontainer内のDocker経由）
- **パッケージ管理**: Poetry
- **Devcontainer**: PostgreSQLサービス付きVS Code devcontainer

### 本番デプロイ
- **サーバー**: Ubuntu 22.04のEC2
- **Webサーバー**: リバースプロキシとしてのNginx
- **アプリサーバー**: カスタム設定のGunicorn
- **データベース**: PostgreSQL 15
- **CI/CD**: developブランチのプッシュとタグ付きリリースでの自動デプロイを行うGitHub Actions
- **デプロイ機能**: 
  - バージョン管理によるロールバック機能
  - デプロイ後のヘルスチェック
  - デプロイ前の自動テスト

### モデル
- **Post**（diary/models.py）: タイトル、内容、タイムスタンプを持つブログ投稿モデル

### 環境変数
開発環境ではデフォルト値を使用、本番環境では以下が必要:
- `SECRET_KEY`: Django秘密キー
- `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`: データベース認証情報
- `DOMAIN_NAME`, `SERVER_IP`: サーバー設定
- `USE_HTTPS`: SSL/TLS設定

### GitHub Actionsワークフロー
- **テストジョブ**: PostgreSQLサービスで全てのプッシュ/PRで実行
- **デプロイジョブ**: developブランチプッシュ、タグプッシュ、または手動ディスパッチでトリガー
- **ロールバックジョブ**: バージョン指定による手動ロールバック機能
- **ヘルスチェック**: デプロイ/ロールバック後の自動検証

## 開発ノート

### 新しいアプリケーションの追加
1. アプリを作成: `poetry run python manage.py startapp <app_name>`
2. `config/settings.py`の`INSTALLED_APPS`に追加
3. `config/urls.py`でURLを設定

### データベーススキーマの変更
モデル変更後は常にマイグレーションを作成:
```bash
poetry run python manage.py makemigrations
poetry run python manage.py migrate
```

### 本番環境の考慮事項
- 静的ファイルはWhiteNoise経由で配信
- デバッグツールバーは本番環境で自動的に無効化
- 本番環境ではファイルとコンソールハンドラーでログ設定
- 本番設定でセキュリティヘッダーを強制

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yukeippi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
