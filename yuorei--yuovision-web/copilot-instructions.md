## yuovision-web

> このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

## 開発コマンド

- `npm run dev` - 開発サーバーを起動 (Next.js)
- `npm run build` - 本番用にアプリケーションをビルド
- `npm start` - 本番サーバーを起動
- `npm run lint` - ESLintでコードリンティングを実行
- `npm run generate` - codegenを使用してスキーマからGraphQLコードを生成

Makefileを使用した代替コマンド:
- `make dev` - `npm run dev` と同じ
- `make build` - `npm run build` と同じ
- `make prod` - ビルドして本番サーバーを起動

## プロジェクトアーキテクチャ

### コア技術
- **フロントエンド**: Next.js 14 (App Router使用)
- **スタイリング**: Tailwindcss と Radix UI コンポーネント
- **GraphQL**: Apollo Client (型生成用のcodegenを使用)
- **動画ストリーミング**: HLS.js による動画再生
- **認証**: NextAuth (beta) とlocalStorageでのトークン管理
- **状態管理**: Zustand
- **ファイルアップロード**: GraphQLファイルアップロード用のApollo upload client

### 主要ディレクトリ構造
```
src/app/
├── apollo/          # Apollo Client設定
├── components/      # 再利用可能なReactコンポーネント
│   ├── ui/         # ベースUIコンポーネント (shadcn/uiスタイル)
│   └── ...         # 機能固有のコンポーネント
├── gql/            # 生成されたGraphQLの型とフック
├── lib/            # ユーティリティ関数と設定
├── model/          # TypeScript型定義
└── providers/      # Reactコンテキストプロバイダー
```

### 重要なファイル
- `codegen.ts` - GraphQLコード生成設定 (スキーマ: http://localhost:8080/query)
- `src/app/apollo/client.ts` - Apollo Clientの認証・アップロードリンク設定
- `src/app/providers/WithApolloProvider.tsx` - Apollo Providerラッパー
- `src/app/layout.tsx` - メンテナンスモード対応のルートレイアウト

### GraphQL統合
- スキーマエンドポイント: `http://localhost:8080/query` (codegen.tsで設定)
- 生成された型とフックは `src/app/gql/` に配置
- 動画ファイル処理のアップロード対応Apollo Clientを使用
- localStorageに保存されたBearerトークンによる認証

### 動画プラットフォーム機能
- カスタムプレーヤーコンポーネントを使用したHLS動画ストリーミング
- GraphQLミューテーションによる動画アップロード機能
- チャンネルベースのコンテンツ整理
- 動画コメントシステム
- ソーシャルシェア機能

### 環境設定
- GraphQLエンドポイントに `NEXT_PUBLIC_API_URL` を使用
- `NEXT_PUBLIC_MAINTENANCE_MODE` でメンテナンスモードを制御
- `NEXT_PUBLIC_GA_ID` でGoogle Analytics統合
- `video-storage.yuorei.com` に対する画像最適化設定

### ビルド設定
- コンテナ化デプロイメント用のNext.js standalone出力
- 本番デプロイメント用のDockerfileを含む
- 外部動画サムネイル用の画像ドメインホワイトリスト

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuorei)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yuorei)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
