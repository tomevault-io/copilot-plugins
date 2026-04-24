## summeryme-ai

> <path>/backend_ts</path>

# work directory

以下のディレクトリで作業すること
<working_dir>
<path>/backend_ts</path>
</working_dir>

# Backend TypeScript Project Rules

## プロジェクト概要
このプロジェクトは、Hono + TypeScript + Prisma + MySQLを使用したバックエンドAPIです。

## 技術スタック
- **フレームワーク**: Hono v4.7.10
- **言語**: TypeScript (ESNext, NodeNext)
- **ORM**: Prisma v6.8.2
- **データベース**: MySQL 8.3
- **パッケージマネージャー**: pnpm
- **開発環境**: Docker Compose
- **コード品質**: ESLint + Prettier

## コーディング規約

### TypeScript
- 厳密な型チェックを使用（strict: true）
- `any`型の使用は最小限に抑制
- 未使用変数は警告として扱う
- ESNext構文を積極的に使用
- verbatimModuleSyntax: trueに従う

### インポート順序
以下の順序でインポートを整理：
1. Node.js組み込みモジュール
2. 外部ライブラリ
3. 内部モジュール（@/で始まるパス）
4. 相対パス（親ディレクトリ）
5. 相対パス（同階層・子ディレクトリ）
6. 型インポート

各グループ間は空行で区切り、アルファベット順に並べる。

### コードフォーマット
- シングルクォート使用
- セミコロン必須
- 行末カンマあり
- 行幅80文字
- インデント2スペース
- アロー関数の括弧は常に使用

### API設計
- RESTful APIの原則に従う
- エラーハンドリングは適切なHTTPステータスコードを使用
- レスポンスは一貫したJSON形式
- 日本語エラーメッセージを使用

### Prisma
- スキーマファイルは`src/prisma/schema.prisma`に配置
- 生成されたクライアントは`src/prisma/generated/prisma`に出力
- マイグレーションファイルは適切な命名規則を使用
- データベース接続は適切にクリーンアップ

### ファイル構造
```
src/
├── index.ts          # エントリーポイント
├── prisma/
│   ├── schema.prisma # Prismaスキーマ
│   ├── migrations/   # マイグレーションファイル
│   └── generated/    # 生成されたPrismaクライアント
├── routes/           # ルートハンドラー（推奨）
├── middleware/       # ミドルウェア（推奨）
├── services/         # ビジネスロジック（推奨）
├── types/           # 型定義（推奨）
└── utils/           # ユーティリティ関数（推奨）
```

## 開発ガイドライン

### 新機能開発時
1. 適切な型定義を作成
2. エラーハンドリングを実装
3. Prismaクエリは効率的に記述
4. テストケースを考慮した実装
5. ログ出力を適切に配置

### データベース操作
- トランザクションが必要な場合は適切に使用
- N+1問題を避けるためincludeやselectを活用
- インデックスを考慮したクエリ設計
- マイグレーションは本番環境を考慮

### セキュリティ
- 入力値の検証を徹底
- SQLインジェクション対策（Prismaが自動対応）
- 適切な認証・認可の実装
- 機密情報の環境変数管理

### パフォーマンス
- 不要なデータの取得を避ける
- 適切なページネーション実装
- キャッシュ戦略の検討
- データベース接続プールの最適化

## 禁止事項
- `any`型の多用
- console.logの本番環境への残存
- ハードコードされた設定値
- 適切でないエラーハンドリング
- 型安全性を損なう実装

## 推奨事項
- 関数型プログラミングの活用
- 純粋関数の作成
- 適切な抽象化レベルの維持
- 可読性の高いコード記述
- 適切なコメントの追加

## 開発コマンド
```bash
# 開発サーバー起動
pnpm dev

# ビルド
pnpm build

# 本番サーバー起動
pnpm start

# リント
pnpm lint

# フォーマット
pnpm format

# Prismaクライアント生成
npx prisma generate

# マイグレーション実行
npx prisma migrate dev --name <migration_name>

# データベースリセット
npx prisma migrate reset

# Docker環境起動
docker compose up -d
```

## 環境設定
- `.env`ファイルでDATABASE_URLを設定
- Docker Composeでローカル開発環境を構築
- MySQL 8.3を使用
- ポート8080でサーバー起動

## github actions

github actionsはリポジトリのルートの.github ディレクトリを利用する

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shuheu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
