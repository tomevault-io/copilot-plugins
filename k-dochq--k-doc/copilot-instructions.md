## k-doc

> This rule explains Next.js conventions and best practices for fullstack development.


# Next.js rules

- Use the App Router structure with `page.tsx` files in route directories.
- Client components must be explicitly marked with `'use client'` at the top of the file.
- Use kebab-case for directory names (e.g., `components/auth-form`) and PascalCase for component
  files.
- Prefer named exports over default exports, i.e. `export function Button() { /* ... */ }` instead
  of `export default function Button() { /* ... */ }`.
- Minimize `'use client'` directives:
  - Keep most components as React Server Components (RSC)
  - Only use client components when you need interactivity and wrap in `Suspense` with fallback UI
  - Create small client component wrappers around interactive elements
- Avoid unnecessary `useState` and `useEffect` when possible:
  - Use server components for data fetching
  - Use URL search params for shareable state
- Use `nuqs` for URL search param state management

## Internationalization (i18n) Rules

### Locale-Aware Navigation

- **Never use raw `useRouter` or `Link`** for internal navigation in internationalized apps
- **Always use `LocaleLink` component** for internal links that need locale prefixing
- **Always use `useLocalizedRouter` hook** instead of raw `useRouter` for programmatic navigation

#### LocaleLink Usage

```typescript
// ✅ Good - Automatic locale handling
import { LocaleLink } from 'shared/ui/locale-link';

<LocaleLink href="/auth/signup">
  회원가입
</LocaleLink>

// ✅ With custom locale
<LocaleLink href="/auth/signup" locale="en">
  Sign Up
</LocaleLink>
```

#### useLocalizedRouter Usage

```typescript
// ✅ Good - Automatic locale prefixing
import { useLocalizedRouter } from 'shared/model/hooks/useLocalizedRouter';

function MyComponent() {
  const router = useLocalizedRouter();

  const handleNavigate = () => {
    router.push('/auth/signup'); // Automatically becomes /en/auth/signup
    router.replace('/dashboard'); // Automatically becomes /en/dashboard
  };

  return <button onClick={handleNavigate}>Navigate</button>;
}

// ❌ Bad - Manual locale handling
import { useRouter, usePathname } from 'next/navigation';
import { DEFAULT_LOCALE, type Locale } from 'shared/config';

function MyComponent() {
  const router = useRouter();
  const pathname = usePathname();

  const currentLocale = (pathname.split('/')[1] as Locale) || DEFAULT_LOCALE;

  const handleNavigate = () => {
    router.push(`/${currentLocale}/auth/signup`); // Manual locale handling
  };
}
```

### Benefits of Locale-Aware Navigation

- **Consistency**: All internal links automatically include correct locale
- **Maintainability**: Locale logic centralized in reusable components
- **Developer Experience**: No need to manually handle locale in every component
- **Error Prevention**: Prevents broken links due to missing locale prefixes

## TypeScript Rules

- **Never use `any` type**: Always provide specific types for better type safety
  - Use `unknown` for truly unknown types that need type guards
  - Use union types like `string | number` for multiple possible types
  - Use generics `<T>` for reusable type-safe functions
  - Use proper interface or type definitions instead of `any`

### Page Props Type Safety

- **Always use Promise types for page props** in Next.js App Router (Next.js 15)
- **`params` and `searchParams` are now Promise types** and must be awaited

#### ✅ Correct Page Props Usage

```typescript
interface PageProps {
  params: Promise<{
    lang: Locale;
    id?: string;
  }>;
  searchParams: Promise<{
    error?: string;
    provider?: string;
  }>;
}

export default async function Page({ params, searchParams }: PageProps) {
  const { lang, id } = await params;
  const resolvedSearchParams = await searchParams;

  // Use lang, id, resolvedSearchParams.error, etc.
}
```

#### ❌ Incorrect Page Props Usage

```typescript
interface PageProps {
  params: {
    lang: Locale;
  };
  searchParams: {
    error?: string;
  };
}

export default async function Page({ params, searchParams }: PageProps) {
  // ❌ This will cause TypeScript errors
  const dict = await getDictionary(params.lang);
  const error = searchParams.error;
}
```

#### Common Build Errors to Avoid

- **Type error**: `Type 'PageProps' does not satisfy the constraint 'PageProps'`
- **Missing properties**: `Types of property 'params' are incompatible`
- **Promise methods**: `Type is missing the following properties: then, catch, finally`

#### Migration Pattern

```typescript
// Before (Next.js 13)
interface Props {
  params: { lang: string };
  searchParams: { error?: string };
}

// After (Next.js 15)
interface Props {
  params: Promise<{ lang: string }>;
  searchParams: Promise<{ error?: string }>;
}

export default async function Page({ params, searchParams }: Props) {
  const { lang } = await params;
  const { error } = await searchParams;
  // ...
}
```

## TanStack Query Best Practices

### Setup and Configuration

- Create a QueryClient wrapper with proper configuration:

```typescript
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

function makeQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000, // 1 minute
        gcTime: 10 * 60 * 1000, // 10 minutes (formerly cacheTime)
      },
    },
  });
}

let browserQueryClient: QueryClient | undefined = undefined;

export function getQueryClient() {
  if (typeof window === 'undefined') {
    // Server: always make a new query client
    return makeQueryClient();
  } else {
    // Browser: make a new query client if we don't already have one
    if (!browserQueryClient) browserQueryClient = makeQueryClient();
    return browserQueryClient;
  }
}
```

- Use QueryClientProvider in a client component wrapper:

```typescript
// components/providers.tsx
'use client'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { useState } from 'react'

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000, // 1 minute
      },
    },
  }))

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

### Server-Side Data Prefetching

- Prefetch data in Server Components and use HydrationBoundary:

```typescript
// app/posts/page.tsx
import { HydrationBoundary, dehydrate } from '@tanstack/react-query'
import { getQueryClient } from '@/lib/query-client'
import { PostsList } from './posts-list'

export default async function PostsPage() {
  const queryClient = getQueryClient()

  await queryClient.prefetchQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  })

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <PostsList />
    </HydrationBoundary>
  )
}
```

- Create typed query functions and hooks:

```typescript
// lib/queries/posts.ts
import { useQuery, useSuspenseQuery } from '@tanstack/react-query';

export interface Post {
  id: number;
  title: string;
  content: string;
}

export async function fetchPosts(): Promise<Post[]> {
  const response = await fetch('/api/posts');
  if (!response.ok) throw new Error('Failed to fetch posts');
  return response.json();
}

export function usePosts() {
  return useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });
}

export function usePostsSuspense() {
  return useSuspenseQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });
}
```

### Client Components Best Practices

- Use `useSuspenseQuery` with Suspense boundaries for better UX:

```typescript
// components/posts-list.tsx
'use client'
import { Suspense } from 'react'
import { usePostsSuspense } from '@/lib/queries/posts'

function PostsContent() {
  const { data: posts } = usePostsSuspense()

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>{post.title}</article>
      ))}
    </div>
  )
}

export function PostsList() {
  return (
    <Suspense fallback={<div>Loading posts...</div>}>
      <PostsContent />
    </Suspense>
  )
}
```

### Query Key Management

- Use consistent, hierarchical query keys:

```typescript
// lib/query-keys.ts
export const queryKeys = {
  posts: ['posts'] as const,
  post: (id: number) => ['posts', id] as const,
  users: ['users'] as const,
  user: (id: string) => ['users', id] as const,
  profile: (userId: string) => ['profile', userId] as const,
} as const;
```

### Mutations and Optimistic Updates

- Handle mutations with proper error handling and optimistic updates:

```typescript
// lib/mutations/posts.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { queryKeys } from '@/lib/query-keys';

export function useCreatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (newPost: Omit<Post, 'id'>) => {
      const response = await fetch('/api/posts', {
        method: 'POST',
        body: JSON.stringify(newPost),
        headers: { 'Content-Type': 'application/json' },
      });
      if (!response.ok) throw new Error('Failed to create post');
      return response.json();
    },
    onSuccess: () => {
      // Invalidate and refetch posts
      queryClient.invalidateQueries({ queryKey: queryKeys.posts });
    },
    onError: (error) => {
      console.error('Failed to create post:', error);
    },
  });
}
```

### Performance Optimization

- Use `enabled` option to prevent unnecessary requests:

```typescript
export function useUserProfile(userId?: string) {
  return useQuery({
    queryKey: queryKeys.profile(userId!),
    queryFn: () => fetchUserProfile(userId!),
    enabled: !!userId, // Only run when userId is available
  });
}
```

- Implement proper loading and error states:

```typescript
function PostComponent() {
  const { data, isLoading, error } = usePosts()

  if (isLoading) return <PostSkeleton />
  if (error) return <ErrorMessage error={error} />
  if (!data) return <EmptyState />

  return <PostsList posts={data} />
}
```

### Route Handler Integration

#### **GET Requests in Server Components**

```typescript
// ✅ Good - Server component for data fetching
export default async function PostsPage() {
  const queryClient = getQueryClient();

  await queryClient.prefetchQuery({
    queryKey: ['posts'],
    queryFn: () => fetch('/api/posts').then(res => res.json()),
  });

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <PostsList />
    </HydrationBoundary>
  );
}
```

#### **Mutations with Route Handlers**

```typescript
// ✅ Good - Client component with mutations
'use client';
import { useMutation, useQueryClient } from '@tanstack/react-query';

function useCreatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreatePostData) =>
      fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      }).then((res) => res.json()),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
}
```

#### **Route Handler Structure**

```typescript
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET() {
  // Handle GET requests
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    // Process POST data
    return NextResponse.json({ success: true, data: result });
  } catch (error) {
    return NextResponse.json({ success: false, error: error.message }, { status: 400 });
  }
}
```

### Data Fetching Strategy

#### **✅ Recommended Approach**

- **GET requests**: Server Components with `useQuery`/`useSuspenseQuery`
- **POST/PUT/DELETE requests**: Client Components with `useMutation`
- **Route Handlers**: For API endpoints that need server-side processing
- **Server Actions**: For simple form submissions (Next.js 13.4+)

#### **❌ Anti-patterns to Avoid**

- **Client components making direct external API calls**
- **Server components handling POST requests**
- **Mixing data fetching patterns inconsistently**

### Common Patterns

- **Background Refetching**: Set appropriate `staleTime` and `gcTime` values
- **Infinite Queries**: Use `useInfiniteQuery` for paginated data
- **Dependent Queries**: Use `enabled` option to chain queries
- **Parallel Queries**: Execute multiple independent queries simultaneously
- **Query Invalidation**: Strategically invalidate related queries after mutations
- **Error Boundaries**: Wrap query components in error boundaries for better error handling

## Next.js 15 Error Handling & Loading States

### ✅ **Comprehensive Error Handling Strategy**

Next.js 15 App Router에서 데이터 페칭과 UI 상태를 안전하게 처리하기 위한 완전한 에러 처리 전략을
구현합니다.

#### **1. Suspense를 활용한 로딩 상태 처리**

```typescript
// features/quick-menu/ui/QuickMenuWrapper.tsx
import { Suspense } from 'react';
import { QuickMenuSkeleton } from './QuickMenuSkeleton';
import { QuickMenuErrorBoundary } from './QuickMenuErrorBoundary';

export function QuickMenuWrapper({ lang }: QuickMenuWrapperProps) {
  return (
    <QuickMenuErrorBoundary lang={lang}>
      <Suspense fallback={<QuickMenuSkeleton />}>
        <QuickMenuContent lang={lang} />
      </Suspense>
    </QuickMenuErrorBoundary>
  );
}
```

#### **2. Error Boundary를 활용한 에러 상태 처리**

```typescript
// features/quick-menu/ui/QuickMenuErrorBoundary.tsx
'use client';

import { Component, type ReactNode } from 'react';

export class QuickMenuErrorBoundary extends Component<Props, State> {
  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('QuickMenu Error Boundary caught an error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <QuickMenuError
          lang={this.props.lang}
          error={this.state.error}
          onRetry={() => this.setState({ hasError: false, error: undefined })}
        />
      );
    }

    return this.props.children;
  }
}
```

#### **3. 구체적인 에러 메시지 제공**

```typescript
// features/quick-menu/api/use-cases/get-categories.ts
export async function getCategories(): Promise<Category[]> {
  try {
    const categories = await prisma.category.findMany({
      where: { categoryType: 'PART', isActive: true },
      orderBy: [{ order: 'asc' }, { createdAt: 'asc' }],
    });
    return categories;
  } catch (error) {
    console.error('Error fetching categories:', error);

    // 더 구체적인 에러 메시지 제공
    if (error instanceof Error) {
      if (error.message.includes('connection')) {
        throw new Error('데이터베이스 연결에 실패했습니다. 잠시 후 다시 시도해주세요.');
      } else if (error.message.includes('timeout')) {
        throw new Error('요청 시간이 초과되었습니다. 잠시 후 다시 시도해주세요.');
      } else {
        throw new Error(`데이터를 불러오는 중 오류가 발생했습니다: ${error.message}`);
      }
    }

    throw new Error('알 수 없는 오류가 발생했습니다. 잠시 후 다시 시도해주세요.');
  }
}
```

#### **4. 빈 데이터 상태 처리**

```typescript
// features/quick-menu/ui/QuickMenu.tsx
export function QuickMenu({ lang, categories }: QuickMenuProps) {
  const partCategories = categories
    .filter((category) => category.categoryType === 'PART')
    .sort((a, b) => (a.order || 0) - (b.order || 0));

  // 빈 데이터 상태 처리
  if (partCategories.length === 0) {
    return <QuickMenuEmpty lang={lang} />;
  }

  return (
    <div className='w-full'>
      {/* 정상 데이터 렌더링 */}
    </div>
  );
}
```

#### **5. 로딩 스켈레톤 UI**

```typescript
// features/quick-menu/ui/QuickMenuSkeleton.tsx
export function QuickMenuSkeleton() {
  return (
    <div className='w-full'>
      <div className='mb-4'>
        <div className='h-6 w-48 bg-gray-200 rounded animate-pulse'></div>
      </div>
      <div className='grid grid-cols-5 gap-4'>
        {Array.from({ length: 10 }).map((_, index) => (
          <div key={index} className='flex flex-col items-center space-y-2 rounded-lg border border-gray-200 bg-white p-3'>
            <div className='h-12 w-12 bg-gray-200 rounded-full animate-pulse'></div>
            <div className='h-4 w-16 bg-gray-200 rounded animate-pulse'></div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### **에러 처리 계층 구조**

```
Error Boundary (최상위)
├── Suspense (로딩 상태)
│   ├── Server Component (데이터 페칭)
│   └── Client Component (UI 렌더링)
│       ├── 정상 데이터
│       ├── 빈 데이터 (Empty State)
│       └── 에러 상태 (Error State)
```

### **다국어 지원 에러 메시지**

```typescript
// features/quick-menu/ui/QuickMenuError.tsx
const messages = {
  ko: {
    errorMessage: '데이터를 불러오는 중 오류가 발생했습니다.',
    retryButton: '다시 시도',
  },
  en: {
    errorMessage: 'An error occurred while loading data.',
    retryButton: 'Retry',
  },
  th: {
    errorMessage: 'เกิดข้อผิดพลาดขณะโหลดข้อมูล',
    retryButton: 'ลองใหม่',
  },
};
```

### **성능 최적화**

- **Streaming**: Suspense를 통한 점진적 로딩
- **Error Recovery**: Error Boundary를 통한 부분적 에러 복구
- **User Experience**: 로딩 스켈레톤으로 자연스러운 로딩 경험
- **Accessibility**: 에러 상태에서도 접근성 유지

### **모니터링 및 디버깅**

```typescript
// 에러 로깅
componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
  console.error('QuickMenu Error Boundary caught an error:', error, errorInfo);
  // 실제 프로덕션에서는 Sentry, LogRocket 등 사용
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/k-dochq) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
