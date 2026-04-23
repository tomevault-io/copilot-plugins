## file-funeral

> このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイドを提供します。

# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイドを提供します。

## プロジェクト概要

file-funeralは、ローカルストレージとクラウドプロバイダー間でファイルをバックアップ・同期するクラウドネイティブなデスクトップアプリケーションです。主な用途は、複数PC間での双方向ファイル同期によるファイル共有です。

**主要機能（計画）:**
- ローカルファイルをクラウドストレージ（AWS S3、GCS、S3互換）にアップロード/バックアップ
- クラウドからローカルへのファイルダウンロード
- ローカルとクラウド間の双方向同期
- 同期状態インジケーター付きファイルリストの視覚的表示
- 「両方保存」方式による競合解決（Dropboxスタイル）

**詳細な要件**: `REQUIREMENTS.md` を参照してください。

---

## 技術スタック

### バックエンド
- **フレームワーク**: Tauri v2.9.5
- **言語**: Rust (edition 2021, 1.77.2以降)
- **非同期ランタイム**: Tokio（計画中）
- **クラウドSDK**:
  - v1.0 (計画): `aws-sdk-s3`
  - v2.0以降 (計画): S3互換、GCS SDK
- **プラグイン**:
  - `tauri-plugin-log` (インストール済み)
  - `tauri-plugin-fs` (計画中、ファイルシステムアクセス)
- **認証情報ストレージ**: `keyring` クレート（計画中、OSキーチェーン）

### フロントエンド
- **ビルドツール**: Vite 7.3.0
- **フレームワーク**: React 19.2.3
- **言語**: TypeScript 5.9.3
- **開発サーバー**: `http://localhost:1420`
- **ビルド出力**: `dist/`ディレクトリ

### 現在インストール済みの依存関係

**Rust (src-tauri/Cargo.toml):**
```toml
[dependencies]
serde_json = "1.0"
serde = { version = "1.0", features = ["derive"] }
log = "0.4"
tauri = { version = "2.9.5", features = [] }
tauri-plugin-log = "2"
```

**Node.js (package.json):**
- React 19.2.3 + React DOM
- TypeScript 5.9.3
- Vite 7.3.0 + @vitejs/plugin-react
- ESLint + TypeScript ESLint
- @tauri-apps/cli 2.9.6

### 計画中の依存関係（v1.0 MVP）

```toml
# 以下は実装フェーズで追加予定
aws-config = { version = "1.1.7", features = ["behavior-version-latest"] }
aws-sdk-s3 = "1.117.0"
tokio = { version = "1", features = ["full"] }
tauri-plugin-fs = "*"
keyring = "*"
anyhow = "*"
thiserror = "*"
chrono = "*"
```

---

## プロジェクト構造

### 現在の構造

```
file-funeral/
├── src-tauri/              # Rustバックエンド
│   ├── src/
│   │   ├── main.rs         # エントリポイント（app_lib::run()を呼び出し）
│   │   └── lib.rs          # メインロジック + ユニットテスト
│   ├── tests/
│   │   └── integration_test.rs  # 統合テスト
│   ├── Cargo.toml          # Rust依存関係
│   ├── tauri.conf.json     # Tauri設定
│   └── build.rs            # ビルドスクリプト
├── src/                    # Reactフロントエンド
│   ├── main.tsx            # Reactエントリポイント
│   ├── App.tsx             # メインコンポーネント
│   └── App.css
├── dist/                   # Viteビルド出力（.gitignore）
├── index.html              # HTMLテンプレート
├── vite.config.ts          # Vite設定
├── tsconfig.json           # TypeScript設定
├── eslint.config.js        # ESLint設定
├── REQUIREMENTS.md         # 詳細要件定義書
├── TODO.md                 # 実装タスク一覧
└── CLAUDE.md              # このファイル
```

### 計画中の構造（v1.0 MVP実装後）

```
src-tauri/src/
├── main.rs                 # エントリポイント
├── lib.rs                  # 公開API + setup
├── commands.rs             # Tauri Commands（フロントエンド ↔ バックエンド通信）
├── storage/                # クラウドストレージ抽象化層
│   ├── mod.rs              # CloudStorageProvider trait定義
│   ├── s3.rs               # AWS S3実装
│   └── types.rs            # FileInfo, FileMetadata等
├── sync/                   # 同期エンジン
│   ├── mod.rs              # 同期ロジック
│   ├── conflict.rs         # 競合解決
│   └── metadata.rs         # メタデータ管理
└── auth/                   # 認証情報管理
    └── mod.rs              # Keyring統合
```

---

## Tauri固有の開発ノート

### アーキテクチャの理解

**main.rs vs lib.rs:**
- `main.rs`: 最小限のエントリポイント。`app_lib::run()`を呼び出すのみ
- `lib.rs`: 実際のアプリケーションロジック、Tauri setupコード、ユニットテストを含む
- この構成により、ライブラリとしてもバイナリとしてもビルド可能

**Tauri Commands:**
- `#[tauri::command]`マクロでRust関数をフロントエンドに公開
- フロントエンドから`invoke('command_name', { args })`で呼び出し
- 非同期処理はasync fnで実装可能
- エラーは`Result<T, E>`で返し、フロントエンドでキャッチ

**テスト構造:**
- `lib.rs`内の`#[cfg(test)] mod tests`: ユニットテスト
- `tests/`ディレクトリ: 統合テスト（外部クレートとしてアプリをテスト）
- `npm test`で両方を実行（`cd src-tauri && cargo test`）

**設定ファイル:**
- `tauri.conf.json`: アプリケーション設定
  - `devUrl`: `http://localhost:1420`（Vite開発サーバー）
  - `frontendDist`: `../dist`（本番ビルド時の静的ファイル）
  - `beforeDevCommand`: `npm run dev`（開発時にViteを起動）
  - `beforeBuildCommand`: `npm run build`（ビルド時にフロントエンドをビルド）

---

## 計画中のアーキテクチャ（v1.0 MVP）

### ストレージ抽象化レイヤー

アプリはクラウドストレージプロバイダーのためのトレイトベースの抽象化を使用します:

```rust
// src-tauri/src/storage/mod.rs (計画中)
pub trait CloudStorageProvider {
    async fn upload(&self, path: &str, data: &[u8], metadata: FileMetadata) -> Result<()>;
    async fn download(&self, path: &str) -> Result<Vec<u8>>;
    async fn list(&self, prefix: &str) -> Result<Vec<FileInfo>>;
    async fn delete(&self, path: &str) -> Result<()>;
    async fn get_metadata(&self, path: &str) -> Result<FileMetadata>;
}

pub struct FileInfo {
    pub path: String,
    pub size: u64,
    pub last_modified: DateTime<Utc>,
    pub etag: Option<String>,
}

pub struct FileMetadata {
    pub size: u64,
    pub last_modified: DateTime<Utc>,
    pub content_type: Option<String>,
    pub etag: Option<String>,
}
```

**実装計画:**
- v1.0: `S3Provider` (AWS S3)
- v2.0: `S3CompatibleProvider` (MinIO、Backblaze B2等)
- v2.5: `GCSProvider` (Google Cloud Storage)

### 同期エンジン

**同期タイミング:**
- アプリ起動時（自動）
- ユーザーが「同期」ボタンをクリックしたとき（手動）
- （オプション）アプリ終了時

**競合解決:**
- 競合検知時、両方のファイルを保存
- クラウド版: `filename.txt`
- ローカル版: `filename (PC名's conflicted copy YYYY-MM-DD).txt`

**削除の扱い:**
- v1.0: 削除を同期（完全同期）
- v2.0以降: ソフト削除（ゴミ箱/復元機能付き）

---

## 開発コマンド

### セットアップ
```bash
# リポジトリクローン後、最初に実行
npm install

# Tauri CLIをグローバルにインストール（推奨）
cargo install tauri-cli
```

### 開発
```bash
# 開発モードで実行（フロントエンド + バックエンドを同時起動）
npm run tauri:dev

# フロントエンドのみ開発サーバー起動（Tauri無し）
npm run dev

# 本番ビルド（デスクトップアプリケーションとしてパッケージ化）
npm run tauri:build
```

### テスト
```bash
# すべてのテストを実行（Rustのテストのみ現在有効）
npm test

# Rustテストのみ（上記と同じ）
npm run test:rust

# 詳細出力付き（printlnやdbg!マクロの出力を表示）
npm run test:rust:verbose

# 特定のテストのみ実行
cd src-tauri && cargo test test_basic

# テストを監視モードで実行（ファイル変更時に自動再実行）
cd src-tauri && cargo watch -x test
```

### Lint
```bash
# すべてのlintを実行（Rust clippy + TypeScript ESLint）
npm run lint

# Rustのみ（clippy、警告をエラーとして扱う）
npm run lint:rs

# TypeScriptのみ（ESLint）
npm run lint:ts

# 自動修正（ESLintの--fix + cargo fmt）
npm run lint:fix

# フォーマット（cargo fmt + ESLint --fix）
npm run format
```

---

## 重要な実装ガイドライン

### 1. ファイル同期
- 常にTokioで非同期操作を使用する
- ネットワーク障害に対する適切なエラーハンドリングを実装
- ファイル整合性チェックにMD5/ETagを使用
- フルパスを持つS3キーを使用してフォルダ構造を保持

### 2. セキュリティ
- 認証情報を**絶対に**平文で保存しない
- `keyring`クレートを使用してOSキーチェーンを利用:
  - macOS: Keychain
  - Windows: Credential Manager
  - Linux: Secret Service API
- 開発用に環境変数（`AWS_ACCESS_KEY_ID`等）をサポート
- すべてのアップロードでS3サーバーサイド暗号化（SSE-S3）を有効化

### 3. ファイル管理
- **保持するメタデータ**: ファイル名、サイズ、更新日時、ハッシュ値
- **無視するメタデータ**: パーミッション、作成日時、所有者
- **デフォルト除外**: `.DS_Store`、`Thumbs.db`、`*.tmp`、`*.swp`
- **特殊ファイル**: シンボリックリンク、ハードリンクはスキップ（警告表示）
- **サイズ制限**: 最大5GB（v1.0）、100MB以上のファイルはプログレスバー表示

### 4. エラーハンドリング
- ユーザーフレンドリーなエラーメッセージを表示
- トラブルシューティング用の詳細なエラーログを記録
- ネットワークタイムアウトとリトライを適切に処理
- 同期操作を黙って失敗させない

### 5. クロスプラットフォーム対応
- クロスプラットフォームパス処理にTauriのpath APIを使用
- Windows、macOS、Linuxでテスト
- プラットフォーム固有の隠しファイルを適切に処理

### 6. Tauri開発のベストプラクティス
- バックエンドのロジックはRustで実装（パフォーマンス、セキュリティ）
- フロントエンドはUI/UXに集中
- Tauri Commandsは粒度を適切に保つ（細かすぎず、粗すぎず）
- 長時間実行される処理はイベントシステムで進捗を通知
- エラーは適切に型付けしてフロントエンドに伝える

---

## 開発ロードマップ

**詳細な実装タスク一覧**: `TODO.md` を参照してください。

### バージョン計画

- **v1.0 (MVP)**: AWS S3対応、基本的な双方向同期機能
- **v1.5**: 除外パターン機能（.gitignore方式）
- **v2.0**: S3互換対応、複数フォルダ選択、ソフト削除
- **v2.5**: Google Cloud Storage対応
- **v3.0**: Azure Blob Storage、クライアントサイド暗号化、高度な機能

### 現在の進捗状況

**フェーズ0: プロジェクトセットアップ** ✅ 完了
- ✅ Tauri v2 + React 19 + TypeScript 5.9 環境構築
- ✅ テストインフラ構築（cargo test、ユニット + 統合テスト）
- ✅ Lintインフラ構築（clippy, ESLint）
- ✅ 開発サーバー動作確認（localhost:1420）
- ✅ ドキュメント整備（REQUIREMENTS.md, TODO.md, CLAUDE.md）

**次のステップ**: `TODO.md` のフェーズ1（バックエンド基盤）を参照
- AWS SDK依存関係の追加
- Storage抽象化層の実装
- S3Provider実装

---

## Git ワークフロー

- **デフォルトブランチ**: `develop`（開発中の機能はここにマージ）
- **メインブランチ**: `main`（リリース用、本番環境向け）
- **フィーチャーブランチ**: `feature/*`パターン（`develop`からブランチを切り、完了後`develop`へマージ）
- **コミットメッセージ**: Conventional Commitsに従う
  - `feat:` 新機能
  - `fix:` バグ修正
  - `docs:` ドキュメント
  - `test:` テスト追加・修正
  - `refactor:` リファクタリング
  - `chore:` ビルド・設定等
- クリーンな履歴を維持（マージコミットを避ける場合はrebaseを使用）

---

## トラブルシューティング

### 開発サーバーが起動しない
- ポート1420が使用中でないか確認: `lsof -i :1420`
- node_modulesを再インストール: `rm -rf node_modules package-lock.json && npm install`
- Rustの依存関係を再ビルド: `cd src-tauri && cargo clean && cd .. && npm run tauri:dev`

### Rustのコンパイルエラー
- Rustのバージョンを確認: `rustc --version`（1.77.2以降が必要）
- Cargo.lockを削除して再ビルド: `cd src-tauri && rm Cargo.lock && cargo build`

### テストが失敗する
- `npm run test:rust:verbose`で詳細な出力を確認
- 個別のテストを実行: `cd src-tauri && cargo test test_basic -- --nocapture`

---

## リソース

### プロジェクトドキュメント
- **要件定義**: `REQUIREMENTS.md`
- **実装タスク一覧**: `TODO.md`
- **README**: `README.md`

### 公式ドキュメント
- **Tauriドキュメント**: https://v2.tauri.app/
- **Tauri Command**: https://v2.tauri.app/develop/calling-rust/
- **Tauri Events**: https://v2.tauri.app/develop/calling-frontend/
- **AWS SDK for Rust**: https://github.com/awslabs/aws-sdk-rust
- **AWS S3 Examples**: https://github.com/awslabs/aws-sdk-rust/tree/main/examples/examples/s3
- **File Systemプラグイン**: https://v2.tauri.app/plugin/file-system
- **React公式ドキュメント**: https://react.dev/
- **Viteドキュメント**: https://vitejs.dev/

### Rustクレート
- **tokio**: https://tokio.rs/
- **serde**: https://serde.rs/
- **keyring**: https://docs.rs/keyring/
- **anyhow**: https://docs.rs/anyhow/
- **thiserror**: https://docs.rs/thiserror/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ohr486) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
