## study-nihongo-with-clip

> **実装を始める前に、必ず`docs/`ディレクトリ配下のドキュメントを確認してください。**


# プロジェクトガイドライン

## 📚 実装前のドキュメント確認

**実装を始める前に、必ず`docs/`ディレクトリ配下のドキュメントを確認してください。**

### 確認すべきドキュメント

1. **[requirements.md](../docs/requirements.md)** - プロジェクトの要件と仕様
2. **[tech-stack.md](../docs/tech-stack.md)** - 使用技術とバージョン情報
3. **[project-structure.md](../docs/project-structure.md)** - ディレクトリ構造と命名規則
4. **[development-guide.md](../docs/development-guide.md)** - 開発ガイドラインとコーディング規約
5. **[progress.md](../docs/progress.md)** - 現在の進捗状況とタスクリスト

### 確認チェックリスト

実装前に以下を確認：
- [ ] 要件定義書で機能要件を確認
- [ ] 技術スタックで使用するライブラリとバージョンを確認
- [ ] プロジェクト構造でファイル配置ルールを確認
- [ ] 開発ガイドラインでコーディング規約を確認
- [ ] 進捗管理で既存の実装状況を確認

---

## ⚡ Next.js 16 ベストプラクティス

**現在のバージョン**: Next.js 16.1.4, React 19.2.3

**注意**: このプロジェクトではReact 19のキャッシュコンポーネントは有効化していません。

### App Routerの使用

```typescript
// ✅ GOOD: App Routerを使用
// app/page.tsx
export default function Page() {
  return <div>Content</div>
}

// ❌ BAD: Pages Routerは使用しない
// pages/index.tsx (使用しない)
```

### Server ComponentsとClient Components

```typescript
// ✅ GOOD: デフォルトはServer Component
// app/components/ServerComponent.tsx
export default function ServerComponent() {
  // サーバー側で実行される
  return <div>Server Content</div>
}

// ✅ GOOD: インタラクティブな場合は'use client'を使用
// app/components/ClientComponent.tsx
'use client'

import { useState } from 'react'

export default function ClientComponent() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

### データフェッチング

```typescript
// ✅ GOOD: Server Componentで直接fetch
// app/movies/page.tsx
async function getMovies() {
  const res = await fetch('https://api.example.com/movies', {
    cache: 'no-store' // または 'force-cache'
  })
  return res.json()
}

export default async function MoviesPage() {
  const movies = await getMovies()
  return <div>{/* レンダリング */}</div>
}

// ❌ BAD: useEffectでfetchしない（Server Componentでは不要）
```

### 画像の最適化

```typescript
// ✅ GOOD: Next.js Imageコンポーネントを使用
import Image from 'next/image'

export default function MovieCard({ poster }: { poster: string }) {
  return (
    <Image
      src={poster}
      alt="Movie poster"
      width={300}
      height={450}
      priority={false} // 必要に応じて
    />
  )
}

// ❌ BAD: 通常のimgタグは使用しない
```

### メタデータ

```typescript
// ✅ GOOD: メタデータをexport
// app/movies/[id]/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Movie Details',
  description: 'Movie information page',
}

export default function MoviePage() {
  return <div>Content</div>
}
```

### ルーティング

```typescript
// ✅ GOOD: ファイルベースルーティング
// app/movies/page.tsx → /movies
// app/movies/[id]/page.tsx → /movies/:id

// ✅ GOOD: 動的ルート
// app/movies/[id]/page.tsx
export default function MoviePage({ params }: { params: { id: string } }) {
  return <div>Movie {params.id}</div>
}
```

### エラーハンドリング

```typescript
// ✅ GOOD: error.tsxでエラーハンドリング
// app/movies/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>エラーが発生しました</h2>
      <button onClick={() => reset()}>再試行</button>
    </div>
  )
}
```

### ローディング状態

```typescript
// ✅ GOOD: loading.tsxでローディング状態を表示
// app/movies/loading.tsx
export default function Loading() {
  return <div>読み込み中...</div>
}
```

### TypeScriptの型安全性

```typescript
// ✅ GOOD: 適切な型定義
interface Movie {
  id: string
  title: string
  releaseDate: string
}

export default function MovieCard({ movie }: { movie: Movie }) {
  return <div>{movie.title}</div>
}

// ❌ BAD: any型の使用を避ける
```

### パフォーマンス最適化

```typescript
// ✅ GOOD: 動的インポートでコード分割
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <p>読み込み中...</p>,
  ssr: false // 必要に応じて
})

// ✅ GOOD: 適切なキャッシュ戦略
const res = await fetch(url, {
  cache: 'force-cache', // 静的データ
  // または
  cache: 'no-store', // 動的データ
  next: { revalidate: 3600 } // ISR
})
```

### 環境変数

```typescript
// ✅ GOOD: 環境変数の使用
// .env.local
// NEXT_PUBLIC_API_URL=https://api.example.com

// app/config.ts
export const API_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000'
```

### スタイリング

```typescript
// ✅ GOOD: Tailwind CSSを使用（プロジェクトで採用）
// app/components/Button.tsx
export default function Button() {
  return (
    <button className="px-4 py-2 bg-blue-500 text-white rounded">
      Click me
    </button>
  )
}

// ✅ GOOD: CSS Modulesも使用可能
// app/components/Button.module.css
```

### 禁止事項

- ❌ `getServerSideProps`や`getStaticProps`は使用しない（App Routerでは不要）
- ❌ `pages/`ディレクトリは使用しない（App Routerを使用）
- ❌ React 19のキャッシュコンポーネントは使用しない（プロジェクト設定で無効化）
- ❌ クライアントコンポーネントで不要な`'use client'`を使用しない
- ❌ Server Componentでブラウザ専用APIを使用しない

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/asoyuta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
