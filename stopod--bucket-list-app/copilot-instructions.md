## bucket-list-app

> このファイルは、人生でやりたいことリストアプリケーションの開発ガイドラインを提供します。

# CLAUDE.md

このファイルは、人生でやりたいことリストアプリケーションの開発ガイドラインを提供します。

## 必須コマンド

### 開発

- `npm run dev` - 開発サーバーを起動
- `npm run build` - 本番用ビルド
- `npm run start` - 本番サーバーを起動
- `npm run typecheck` - TypeScript型チェックを実行

### テスト・品質管理

- `npm test` - テスト実行
- `npm run test:ui` - UI付きテスト実行
- `npm run test:coverage` - カバレッジ測定

### Docker開発ワークフロー

```bash
# 開発時（DBのみDocker、アプリはローカル）
docker compose -f compose.dev.yaml up -d
npm run dev

# 本番時（全部Docker）
docker compose up -d
```

## CI/CD パイプライン

### GitHub Actions ワークフロー

基本的なCI設定により、以下を自動実行：

#### CI - `.github/workflows/ci.yml`

**トリガー**: main・developブランチへのpush、PRの作成・更新
**実行内容**:

- Node.js 18.x, 20.x でのマトリックステスト
- TypeScript型チェック
- 全テスト実行
- ビルド確認
- Prettier・ESLint（参考情報として実行）

## アーキテクチャ概要

人生でやりたいことリストを管理するWebアプリケーション：

- **フロントエンド**: React Router v7（SSR有効）、TailwindCSS、shadcn-ui
- **バックエンド**: Node.js with TypeScript
- **データベース**: Supabase PostgreSQL
- **認証**: Supabase Auth
- **デプロイ**: Dockerコンテナ化

### アーキテクチャパターン

- **Clean Architecture**: Repository Pattern + Service Layer
- **Dependency Injection**: Factory Pattern によるDI
- **Type Safety**: TypeScript による型安全性
- **SSR**: React Router v7 による Server-Side Rendering
- **Functional Programming**: Result型による安全なエラーハンドリング

### プログラミングアプローチ

#### 関数型アプローチ

- 関数型Service層（`bucket-list.service.ts`）
- Result<T, E> による型安全なエラーハンドリング
- 純粋なビジネスロジック関数（`business-logic.ts`）
- 関数合成とコンビネーター活用
- hooks基底パターン統一（`use-operation-base.ts`）による重複削除

**実装方針**: 関数型アプローチで実装

## 開発ガイドライン

### コードスタイル

1. **Repository Pattern 必須**

   - データアクセス層の抽象化
   - Supabaseクライアントとの密結合を回避
   - テスタビリティの向上

2. **型安全性**

   - 全てのAPIレスポンスに型定義
   - フォームデータの型定義
   - Database型定義の活用

3. **エラーハンドリング**
   - Repository層でのエラー変換
   - ユーザーフレンドリーなエラーメッセージ
   - 適切なHTTPステータスコード
   - **Result型使用**: Result<T, E>による型安全なエラーハンドリング

### ファイル構成規則

```
app/
├── features/bucket-list/           # 機能単位のモジュール
│   ├── components/                 # UI コンポーネント
│   ├── repositories/              # データアクセス層
│   │   └── bucket-list.repository.ts  # 関数型Repository（Result型）
│   ├── services/                  # ビジネスロジック層
│   │   └── bucket-list.service.ts      # 関数型Service（Result型）
│   ├── lib/                       # ファクトリ等のユーティリティ
│   │   ├── business-logic.ts      # 純粋なビジネスロジック関数
│   │   └── service-factory.ts     # 関数合成DIファクトリ
│   └── types.ts                   # 型定義
├── shared/                        # 共通モジュール
│   ├── layouts/                   # レイアウトコンポーネント
│   ├── types/                     # 共通型定義
│   │   ├── result.ts              # Result<T, E>型定義
│   │   └── errors.ts              # ドメイン別エラー型
│   ├── utils/                     # 共通ユーティリティ
│   │   └── result-helpers.ts      # Result操作ヘルパー関数
│   └── hooks/                     # 共通hooks
│       ├── use-operation-base.ts   # 非同期操作基底クラス
│       ├── use-result-operation.ts # Result対応hooks
│       └── use-async-operation.ts  # 非同期操作hooks
└── routes/                        # ページコンポーネント
```

### データベース設計原則

1. **テーブル設計**

   - `profiles` テーブル: auth.users の代替（非推奨回避）
   - `categories` テーブル: 事前定義カテゴリ
   - `bucket_items` テーブル: やりたいこと項目

2. **RLS (Row Level Security)**

   - 全テーブルでRLS有効化必須
   - ユーザー毎のデータ分離
   - 公開データのセキュリティ制御

3. **インデックス戦略**
   - 検索・フィルタ対象カラムにインデックス
   - 複合インデックスの活用

## 機能仕様

### 実装機能

#### 基本CRUD操作

- やりたいこと項目の作成・編集・削除・一覧表示
- カテゴリ、優先度、ステータス、期限設定
- 公開/非公開設定

#### 高度な機能

- 検索・フィルタ・ソート機能
- リアルタイムステータス変更
- 達成率計算・可視化
- カテゴリ別進捗表示
- 公開リスト閲覧機能
- ダッシュボード（統計表示）

#### UI/UX

- レスポンシブデザイン
- カード形式表示
- カテゴリ別グループ化
- 削除確認ダイアログ
- プログレスバー表示
- ランディングページ（ヒーローセクション、機能紹介、デモ統計）
- 認証状態別ナビゲーション（自動ダッシュボードリダイレクト）

### データ構造

#### カテゴリ (categories)

- 旅行・観光、スキル習得・学習、体験・チャレンジ
- 人間関係、健康・フィットネス、創作・芸術
- キャリア・仕事、狂気、その他

#### 優先度 (priority)

- `high`: 高
- `medium`: 中
- `low`: 低

#### ステータス (status)

- `not_started`: 未着手
- `in_progress`: 進行中
- `completed`: 完了

#### 期限タイプ (due_type)

- `specific_date`: 具体的日付
- `this_year`: 今年中
- `next_year`: 来年中
- `unspecified`: 未定

## セキュリティ考慮事項

### 認証・認可

- Supabase Auth による認証
- SSRでの認証状態管理
- 適切なリダイレクト処理

### データ保護

- RLS による行レベルセキュリティ
- XSS対策（入力値サニタイズ）
- CSRF対策（SameSite Cookie）

### プライバシー

- 個人データの適切な取り扱い
- 公開設定の尊重
- ユーザー同意に基づく機能提供

## パフォーマンス最適化

### データベース

- 適切なインデックス設計
- クエリの最適化
- N+1問題の回避

### フロントエンド

- コンポーネントの再利用
- 適切な状態管理
- 画像最適化

## 技術的負債・注意事項

### RLS (Row Level Security)

- **現在の状況**: bucket_itemsテーブルでRLS無効化（実用性重視）
- **セキュリティ**: アプリケーション層での認証チェックで代替
- **将来対応**: 必要に応じてRLS有効化への移行を検討

### エラーハンドリング

- 現在はコンソールログ中心
- 本番環境では適切なログシステムが必要

### テスト

- **テストインフラ**: Vitest + @testing-library/react
- **基本テスト実装**: 118テスト実装
  - 非同期操作hooks（use-async-operation、use-result-operation）
  - 認証ガード（auth-guard-simple）
  - Repository層・Service層・ビジネスロジック層
- **Result型テスト**: 型安全なエラーハンドリングの検証

## テスト規約

### テストコード作成ルール

#### 1. テストケース記述規則

- **日本語必須**: 全てのテストケース名は日本語で記述する
- **条件＋期待値形式**: 「〜の場合、〜であること」の形式で記述する
- **具体的な観点**: 確認観点と期待される結果を明確に示す

```typescript
// ✅ Good
it("ユーザーIDを指定した場合、そのユーザーのバケットリスト項目が取得できること", async () => {
  // テストコード
});

it("空の統計データを渡した場合、0%表示で適切に処理されること", () => {
  // テストコード
});

// ❌ Bad
it("should return user bucket items", async () => {
  // テストコード
});

it("handles empty data", () => {
  // テストコード
});
```

#### 2. テスト対象の優先順位

1. **ビジネスロジック**: Service層（関数型・従来型両方）、Repository層
2. **純粋関数**: business-logic.ts内の計算・変換関数
3. **Result型処理**: エラーハンドリングとResult型変換
4. **カスタムコンポーネント**: 独自実装のUIコンポーネント
5. **統合テスト**: コンポーネント間の連携
6. **外部ライブラリのコンポーネント**: テスト不要（shadcn-ui等）

#### 3. テストファイル構成

- `__tests__` ディレクトリ内に配置
- ファイル名: `[対象ファイル名].test.ts(x)`
- Service層、Component層毎にディレクトリ分割

#### 4. Result型テストの書き方

```typescript
// ✅ Good: Result型の成功・失敗両方をテスト
it("有効なデータの場合、Result<Success>で新しい項目が返されること", async () => {
  const result = await createBucketItem(repository)(validData);

  expect(isSuccess(result)).toBe(true);
  if (isSuccess(result)) {
    expect(result.data).toEqual(expectedItem);
  }
});

it("無効なデータの場合、Result<Failure>で返されること", async () => {
  const result = await createBucketItem(repository)(invalidData);

  expect(isFailure(result)).toBe(true);
  if (isFailure(result)) {
    expect(result.error.type).toBe("ValidationError");
  }
});
```

### テスト実行コマンド

```bash
npm test          # 通常実行
npm run test:ui   # UI付き実行
npm run test:coverage  # カバレッジ測定
```

## 環境変数

```bash
# Supabase設定
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key  # サーバーサイド認証用（必須）

# React Router設定
NODE_ENV=development|production
```

**重要**: `SUPABASE_SERVICE_ROLE_KEY`はサーバーサイド認証に必要です。Supabaseダッシュボード → プロジェクト設定 → APIから取得してください。

## 今後のメンテナンス指針

### 即座実行（緊急）

1. **コード品質向上**: TypeScriptエラー修正、不要ファイル削除（Phase 1）
2. **構造最適化**: 重複コンポーネント統合、型定義整理（Phase 2）

### 継続的改善

1. **セキュリティ強化**: 必要に応じてRLS有効化、監査ログ追加
2. **テスト追加**: 品質保証のためのテストコード実装（Phase 3）
3. **モニタリング**: エラー追跡、パフォーマンス監視システム導入
4. **機能拡張**: ソーシャル機能、エクスポート機能、通知機能

### 関連ドキュメント

- アーキテクチャ詳細: `docs/development/architecture.md`
- 認証システム: `docs/development/authentication.md`
- プロジェクト状況: `docs/project/status.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stopod)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/stopod)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
