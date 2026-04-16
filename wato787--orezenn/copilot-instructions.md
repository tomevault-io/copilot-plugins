## orezenn

> - **必須**: 1つのコンポーネントは1つの責任のみ


# コンポーネント設計ルール

## 🎨 基本設計原則

### 単一責任の原則
- **必須**: 1つのコンポーネントは1つの責任のみ
- **例**: `ArticleCard` = 記事カードの表示のみ
- **分割**: 複雑になったら小さなコンポーネントに分割

### Props設計
- **必須**: Props の型定義を明確にする
- **推奨**: デフォルトpropsは使わず、オプショナル型で定義
- **命名**: 明確で理解しやすい名前

```typescript
// ✅ Good
interface ArticleCardProps {
  article: Article;
  onClick?: (id: string) => void;
  showAuthor?: boolean;
  className?: string;
}

const ArticleCard = ({ article, onClick, showAuthor = true, className }: ArticleCardProps) => {
  // ...
};

// ❌ Bad
const ArticleCard = ({ data, click, show }: any) => { // 型定義なし、曖昧な命名
  // ...
};
```

## 🎯 Tailwind CSS ルール

### クラス使用方針
- **必須**: Tailwind CSS v4のユーティリティクラスを使用
- **推奨**: レスポンシブデザイン（sm:, md:, lg:）
- **推奨**: 状態変化クラス（hover:, focus:）

```typescript
// ✅ Good
<button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded transition duration-300 md:px-6 lg:px-8">
  ボタン
</button>

// ❌ Bad
<button style={{ backgroundColor: 'blue' }}> // インラインスタイルは避ける
  ボタン
</button>
```

### shadcn/ui使用方針
- **必須**: shadcn/uiコンポーネントを優先的に使用
- **設定**: `components.json`でプロジェクト設定管理
- **カスタマイズ**: CSS変数（--background, --foregroundなど）で統一テーマ
- **コンポーネント追加**: `npx shadcn@latest add [component]`

### カスタムスタイル
- **原則**: shadcn/ui → Tailwind → CSS Modulesの順で検討
- **推奨**: CSS変数とTailwindの組み合わせ活用

## 📱 レスポンシブ対応

### ブレークポイント
- **sm**: 640px以上（タブレット）
- **md**: 768px以上（小型ノートPC）
- **lg**: 1024px以上（デスクトップ）
- **xl**: 1280px以上（大型ディスプレイ）

```typescript
// ✅ Good - レスポンシブ対応
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {articles.map(article => (
    <ArticleCard key={article.id} article={article} />
  ))}
</div>
```

## 🏗️ コンポーネント構成

### ディレクトリ構造
```
src/components/
├── layout/           # レイアウト関連（作成済み）
│   ├── Header.tsx    ✅ モダンなナビゲーション
│   ├── Footer.tsx    ✅ リンク集付きフッター
│   └── Layout.tsx    ✅ 統合レイアウト
├── ui/              # shadcn/ui基本コンポーネント（作成済み）
│   ├── button.tsx    ✅ shadcn/ui Button
│   ├── card.tsx      ✅ shadcn/ui Card
│   └── navigation-menu.tsx ✅ shadcn/ui Navigation
└── features/        # 機能別コンポーネント（今後作成）
    ├── ArticleCard.tsx
    ├── ArticleList.tsx
    └── CategoryFilter.tsx
```

### ファイル命名
- **コンポーネント**: PascalCase（例: `ArticleCard.tsx`）
- **インデックス**: 必要に応じて`index.ts`でre-export

```typescript
// components/features/index.ts
export { ArticleCard } from './ArticleCard';
export { ArticleList } from './ArticleList';
export { CategoryFilter } from './CategoryFilter';
```

## 🔄 状態管理

### ローカル状態
- **基本**: `useState`を使用
- **複雑**: `useReducer`を検討

```typescript
// ✅ シンプルな状態
const [isOpen, setIsOpen] = useState(false);

// ✅ 複雑な状態
const [state, dispatch] = useReducer(modalReducer, initialState);
```

### グローバル状態
- **サーバー状態**: TanStack Query
- **クライアント状態**: props drilling → Context → Zustand（将来検討）

## 📋 エラーハンドリング

### エラー境界
- **推奨**: Error Boundaryコンポーネントの実装（今後）
- **フォールバック**: 適切なエラー表示

### ローディング状態
- **必須**: 非同期処理にはローディング表示
- **推奨**: Skeleton UIを活用

```typescript
// ✅ Good
const ArticleList = () => {
  const { data: articles, isLoading, isError } = useQuery({
    queryKey: ['articles'],
    queryFn: fetchArticles,
  });

  if (isLoading) return <ArticleSkeleton />;
  if (isError) return <ErrorMessage />;

  return (
    <div className="space-y-4">
      {articles?.contents.map(article => (
        <ArticleCard key={article.id} article={article} />
      ))}
    </div>
  );
};
```

## 🎭 アクセシビリティ

### 基本対応
- **必須**: 適切なセマンティックHTML
- **推奨**: ARIA属性の適切な使用
- **推奨**: キーボードナビゲーション対応

```typescript
// ✅ Good
<button
  aria-label="記事を読む"
  onClick={() => navigate(`/articles/${article.slug}`)}
  className="focus:ring-2 focus:ring-blue-500"
>
  続きを読む
</button>
```

---

**最終更新**: 2024年7月
# コンポーネント設計ルール

## 🎨 基本設計原則

### 単一責任の原則
- **必須**: 1つのコンポーネントは1つの責任のみ
- **例**: `ArticleCard` = 記事カードの表示のみ
- **分割**: 複雑になったら小さなコンポーネントに分割

### Props設計
- **必須**: Props の型定義を明確にする
- **推奨**: デフォルトpropsは使わず、オプショナル型で定義
- **命名**: 明確で理解しやすい名前

```typescript
// ✅ Good
interface ArticleCardProps {
  article: Article;
  onClick?: (id: string) => void;
  showAuthor?: boolean;
  className?: string;
}

const ArticleCard = ({ article, onClick, showAuthor = true, className }: ArticleCardProps) => {
  // ...
};

// ❌ Bad
const ArticleCard = ({ data, click, show }: any) => { // 型定義なし、曖昧な命名
  // ...
};
```

## 🎯 Tailwind CSS ルール

### クラス使用方針
- **必須**: Tailwind CSS v4のユーティリティクラスを使用
- **推奨**: レスポンシブデザイン（sm:, md:, lg:）
- **推奨**: 状態変化クラス（hover:, focus:）

```typescript
// ✅ Good
<button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded transition duration-300 md:px-6 lg:px-8">
  ボタン
</button>

// ❌ Bad
<button style={{ backgroundColor: 'blue' }}> // インラインスタイルは避ける
  ボタン
</button>
```

### shadcn/ui使用方針
- **必須**: shadcn/uiコンポーネントを優先的に使用
- **設定**: `components.json`でプロジェクト設定管理
- **カスタマイズ**: CSS変数（--background, --foregroundなど）で統一テーマ
- **コンポーネント追加**: `npx shadcn@latest add [component]`

### カスタムスタイル
- **原則**: shadcn/ui → Tailwind → CSS Modulesの順で検討
- **推奨**: CSS変数とTailwindの組み合わせ活用

## 📱 レスポンシブ対応

### ブレークポイント
- **sm**: 640px以上（タブレット）
- **md**: 768px以上（小型ノートPC）
- **lg**: 1024px以上（デスクトップ）
- **xl**: 1280px以上（大型ディスプレイ）

```typescript
// ✅ Good - レスポンシブ対応
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {articles.map(article => (
    <ArticleCard key={article.id} article={article} />
  ))}
</div>
```

## 🏗️ コンポーネント構成

### ディレクトリ構造
```
src/components/
├── layout/           # レイアウト関連（作成済み）
│   ├── Header.tsx    ✅ モダンなナビゲーション
│   ├── Footer.tsx    ✅ リンク集付きフッター
│   └── Layout.tsx    ✅ 統合レイアウト
├── ui/              # shadcn/ui基本コンポーネント（作成済み）
│   ├── button.tsx    ✅ shadcn/ui Button
│   ├── card.tsx      ✅ shadcn/ui Card
│   └── navigation-menu.tsx ✅ shadcn/ui Navigation
└── features/        # 機能別コンポーネント（今後作成）
    ├── ArticleCard.tsx
    ├── ArticleList.tsx
    └── CategoryFilter.tsx
```

### ファイル命名
- **コンポーネント**: PascalCase（例: `ArticleCard.tsx`）
- **インデックス**: 必要に応じて`index.ts`でre-export

```typescript
// components/features/index.ts
export { ArticleCard } from './ArticleCard';
export { ArticleList } from './ArticleList';
export { CategoryFilter } from './CategoryFilter';
```

## 🔄 状態管理

### ローカル状態
- **基本**: `useState`を使用
- **複雑**: `useReducer`を検討

```typescript
// ✅ シンプルな状態
const [isOpen, setIsOpen] = useState(false);

// ✅ 複雑な状態
const [state, dispatch] = useReducer(modalReducer, initialState);
```

### グローバル状態
- **サーバー状態**: TanStack Query
- **クライアント状態**: props drilling → Context → Zustand（将来検討）

## 📋 エラーハンドリング

### エラー境界
- **推奨**: Error Boundaryコンポーネントの実装（今後）
- **フォールバック**: 適切なエラー表示

### ローディング状態
- **必須**: 非同期処理にはローディング表示
- **推奨**: Skeleton UIを活用

```typescript
// ✅ Good
const ArticleList = () => {
  const { data: articles, isLoading, isError } = useQuery({
    queryKey: ['articles'],
    queryFn: fetchArticles,
  });

  if (isLoading) return <ArticleSkeleton />;
  if (isError) return <ErrorMessage />;

  return (
    <div className="space-y-4">
      {articles?.contents.map(article => (
        <ArticleCard key={article.id} article={article} />
      ))}
    </div>
  );
};
```

## 🎭 アクセシビリティ

### 基本対応
- **必須**: 適切なセマンティックHTML
- **推奨**: ARIA属性の適切な使用
- **推奨**: キーボードナビゲーション対応

```typescript
// ✅ Good
<button
  aria-label="記事を読む"
  onClick={() => navigate(`/articles/${article.slug}`)}
  className="focus:ring-2 focus:ring-blue-500"
>
  続きを読む
</button>
```

---

**最終更新**: 2024年7月

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wato787) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
