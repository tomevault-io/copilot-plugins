## performance-optimization

> - **ALWAYS** use `useCallback` for functions passed as props

# Performance Optimization & Caching (MANDATORY)

## ⚡ **React Performance (MUST OPTIMIZE)**

- **ALWAYS** use `useCallback` for functions passed as props
- **ALWAYS** use `useMemo` for expensive calculations
- **ALWAYS** use `React.memo` for components that re-render frequently
- **NEVER** create new objects/arrays in render without memoization

## 🗄️ **React Query Caching (CRITICAL)**

Based on [src/lib/query-provider.tsx](mdc:src/lib/query-provider.tsx):

- **ALWAYS** use React Query for server state management
- **ALWAYS** implement proper cache invalidation strategies
- **ALWAYS** set appropriate `staleTime` and `gcTime` values
- **NEVER** bypass React Query for data fetching

```typescript
// ✅ CORRECT - Proper React Query usage
const { data: posts } = useQuery({
    queryKey: ['posts', page, limit],
    queryFn: () => getPosts({ page, limit }),
    staleTime: 5 * 60 * 1000, // 5 minutes
    gcTime: 10 * 60 * 1000, // 10 minutes
    refetchOnWindowFocus: false,
});

// ❌ WRONG - Bypassing React Query
const [posts, setPosts] = useState([]);
useEffect(() => {
    fetch('/api/posts')
        .then((res) => res.json())
        .then(setPosts);
}, []);
```

## 🚫 **FORBIDDEN Performance Practices**

- **NEVER** create functions inside render without useCallback
- **NEVER** create new objects/arrays in render without useMemo
- **NEVER** use useEffect for data fetching when React Query is available
- **NEVER** implement custom caching when React Query handles it
- **NEVER** use forceUpdate or similar anti-patterns

## ✅ **REQUIRED Optimization Practices**

- **ALWAYS** implement proper dependency arrays in useEffect
- **ALWAYS** use React.memo for expensive components
- **ALWAYS** implement proper error boundaries
- **ALWAYS** use lazy loading for large components
- **ALWAYS** optimize images with Next.js Image component

## 📊 **Caching Strategy**

- **ALWAYS** use consistent query keys for related data
- **ALWAYS** implement optimistic updates for better UX
- **ALWAYS** handle loading and error states properly
- **ALWAYS** use background refetching for fresh data

description:
globs:
alwaysApply: true

---

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
