## my-db-site

> 個人サイト。Notionをデータソースとした記事・ギャラリー・ツール群を提供するNext.jsアプリ。

# Haruhate (my-db-site)

個人サイト。Notionをデータソースとした記事・ギャラリー・ツール群を提供するNext.jsアプリ。

## 技術スタック

- **フレームワーク**: Next.js (Pages Router) + TypeScript
- **UI**: Chakra UI + Framer Motion
- **データソース**: Notion API
- **テスト**: Vitest + @testing-library/react
- **コンポーネント文書化**: Storybook
- **パッケージ管理**: yarn

## 主要コマンド

```bash
yarn dev          # 開発サーバー起動 (localhost:3000)
yarn build        # 本番ビルド（Vercelデプロイ前の確認に使う）
yarn lint         # ESLint（Errorがあるとvercel buildが失敗する）
yarn test         # Vitest（coverage付き）
```

> `typecheck` スクリプトはない。型チェックは `yarn build` で代替する。

## ディレクトリ構造

```
src/
  pages/          # Next.js Pages Router（ルーティング）
    api/          # APIルート
    gallery/      # ギャラリー系ページ
    article/      # 記事ページ
  features/       # 機能単位のモジュール（コンポーネント・hooks・utils・型を同梱）
    gallery/
      tool/       # ツール系フィーチャー（es-counter など）
      game/
      manga/
  components/     # 複数フィーチャーをまたぐ共通UIコンポーネント
  hooks/          # 共通カスタムフック
  lib/            # 外部サービス連携（notion・contact）
  utils/          # 汎用ユーティリティ関数
  assets/         # 静的データ・型定義
docs/
  ideas/          # 機能アイデアメモ（作業開始前の下書き）
.steering/        # 作業単位の計画・タスクリスト（スペック駆動開発用）
```

## アーキテクチャの原則

- **features/ が主役**: 機能ごとに `components/`, `hooks/`, `utils/`, `types/` を同梱する
- **pages/ は薄く**: ルーティングと data fetching のみ。ロジックは features/ へ
- **components/ は汎用のみ**: 特定機能に依存するものは features/ に置く

## ギャラリーツールの追加方法

新しいツール `foo` を追加する場合:

```
src/features/gallery/tool/foo/   ← フィーチャー実装
src/pages/gallery/tool/foo/      ← ページ（features/を呼び出すだけ）
```

`src/pages/gallery/tool/index.tsx` の `items` 配列にエントリを追加する。

## スペック駆動開発フロー

```
docs/ideas/ にアイデアメモを書く
  ↓
/setup-project で6つの永続ドキュメントを生成（docs/配下）
  ↓
/add-feature [機能名] で実装を自動化
  ↓
.steering/[日付]-[機能名]/ にタスクリストと振り返りが残る
```

## デプロイ

Vercelに自動デプロイ。`yarn build` が通らないとデプロイ失敗になる。
ESLint の Error（Warningは許容）はビルドを落とすため、push前に `yarn lint` を確認する。

---
> Source: [TaiyoNawa/my-db-site](https://github.com/TaiyoNawa/my-db-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
