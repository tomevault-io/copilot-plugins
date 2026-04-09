## shuitnet-sendemail

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

ShuitNet.SendEmailは、コマンドラインベースのメール送信ツールです。MailKitを使用してSMTP経由でメールを送信し、設定ファイルベースのユーザー管理機能を提供します。

## アーキテクチャ

### 主要コンポーネント

- **Program.cs**: エントリーポイント。System.CommandLineを使用してCLIインターフェースを実装
- **SendEmail.cs**: メール送信の核心ロジック。MailKitクライアントを使用してSMTP接続とメール送信を処理
- **Settings.cs**: 設定管理クラス。JSONファイルからユーザー設定を読み込み、新しいユーザーの追加機能
- **UserSetting.cs**: SMTP設定のデータ構造
- **SendException.cs**: カスタム例外クラス
- **SendState.cs**: エラー状態の列挙型
- **PasswordManager.cs**: パスワード暗号化・復号化機能
- **TemplateManager.cs**: メールテンプレート管理機能。JSON形式でテンプレートを保存・読み込み、変数置換機能
- **EmailTemplate.cs**: メールテンプレートのデータ構造。件名・本文・変数リストを含む

### 設定システム

- 設定ファイルパス: `~/.shuitNet/sendemail/conf.d/{domain}.json` （ユーザーディレクトリベース）
- 設定はドメイン単位でファイルが分かれ、メールアドレスをキーとしたJSON形式
- ディレクトリが存在しない場合は自動作成される
- 実行ユーザーごとに独立した設定ファイルを使用
- パスワードは暗号化されて保存される（Windows: DPAPI、Linux: AES with user-specific key）

### テンプレートシステム

- テンプレートファイルパス: `~/.shuitNet/sendemail/templates/{name}.json`
- テンプレートには件名・本文・変数リストが含まれ、`{{変数名}}`形式で変数置換可能
- テンプレート作成・削除・一覧表示機能をCLIから利用可能
- テンプレートファイルは作成・更新日時を自動管理

### ログ設定

- NLogを使用
- 設定ファイル: `nlog.config`
- デバッグモードとリリースモードで異なるログレベル
- ファイル出力、アーカイブ機能付き

## 開発時のコマンド

### ビルド
```bash
dotnet build
```

### プロジェクトの実行
```bash
dotnet run --project ShuitNet.SendEmail
```

### リリースビルド
```bash
dotnet build -c Release
```

### 発行（Windows x64）
```bash
dotnet publish -c Release -r win-x64 --self-contained
```

### 発行（Linux x64）
```bash
dotnet publish -c Release -r linux-x64 --self-contained
```

## 主要機能

### メール送信
```bash
sendemail --to recipient@example.com --from sender@example.com --subject "テスト" --body "メール本文"
```

### ユーザー追加（管理者権限が必要）
```bash
sendemail add-user user@example.com --smtp-server smtp.example.com --smtp-port 587 --smtp-ssl true --smtp-username user --smtp-password pass
```

### テンプレート管理
```bash
# テンプレート作成
sendemail template create welcome --subject "Welcome {{name}}" --body "Hello {{name}}, welcome to our service!"

# テンプレート一覧
sendemail template list

# テンプレート削除
sendemail template delete welcome

# テンプレートを使用してメール送信
sendemail --template welcome --vars "name=John" --from sender@example.com --to recipient@example.com
```

## 依存関係

- **MailKit 4.10.0**: SMTP/IMAP/POP3クライアント
- **System.CommandLine 2.0.0-beta4**: CLIフレームワーク
- **NLog 5.3.4**: ログフレームワーク
- **Microsoft.Extensions.DependencyInjection 9.0.1**: DIコンテナ
- **Microsoft.Extensions.Logging 9.0.1**: ログ抽象化
- **NLog.Extensions.Logging 5.3.15**: NLogのMicrosoft.Extensions.Logging統合
- **System.Security.Cryptography.ProtectedData 9.0.7**: Windowsでのパスワード暗号化

## 注意点

- デバッグモードでは固定のテストパラメータが設定される（Program.cs:14-23）
- 設定ファイルの読み込みに失敗した場合は適切なエラーハンドリングが必要
- SMTP認証エラー、接続エラーなど、詳細なエラー状態を`SendState`で管理
- MailKitの例外を適切にキャッチして`SendException`にラップする
- パスワードは自動的に暗号化されるため、設定ファイルの手動編集時は注意が必要
- 暗号化されたパスワードは同一ユーザー・同一マシンでのみ復号化可能
- 実行ファイル名は`sendemail.exe`（AssemblyName設定により）
- .NET 8.0をターゲットフレームワークとして使用
- 添付ファイルはHTMLメール形式で送信される
- テンプレート変数は`{{変数名}}`形式で、大文字小文字を区別する

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shuitshuit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shuitshuit)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
