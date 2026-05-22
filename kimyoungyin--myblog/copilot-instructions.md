## react-hooks-infinite-loop-prevention

> // ❌ WRONG - Object/Function in dependency array

# React Hooks Infinite Loop Prevention Rules

## 🚨 Critical Anti-Patterns to Avoid

### 1. **Dependency Array Issues**

```typescript
// ❌ WRONG - Object/Function in dependency array
useEffect(() => {
    // effect logic
}, [someObject, someFunction, { key: 'value' }]);

// ✅ CORRECT - Primitive values only
useEffect(() => {
    // effect logic
}, [primitiveValue, stableReference]);
```

### 2. **State Updates in useEffect**

```typescript
// ❌ WRONG - setState in useEffect with state dependency
const [user, setUser] = useState(null);
useEffect(() => {
    if (profile) {
        setUser(profile); // This can cause infinite loops
    }
}, [profile, setUser]); // setUser changes trigger re-renders

// ✅ CORRECT - Use useCallback or move logic outside
const setUserStable = useCallback((user) => {
    setUser(user);
}, []);

useEffect(() => {
    if (profile) {
        setUserStable(profile);
    }
}, [profile, setUserStable]);
```

### 3. **Object References in Dependencies**

```typescript
// ❌ WRONG - New object created on every render
useEffect(() => {
    // effect logic
}, [{ id: 1, name: 'test' }]); // New object every time

// ✅ CORRECT - Use useMemo or extract values
const stableObject = useMemo(() => ({ id: 1, name: 'test' }), []);
useEffect(() => {
    // effect logic
}, [stableObject]);

// OR extract specific values
useEffect(() => {
    // effect logic
}, [object.id, object.name]);
```

## 🔧 Best Practices for State Management

### 1. **Separate Data Fetching from State Updates**

```typescript
// ✅ GOOD - React Query for data, Zustand for state
export function useAuth() {
    // Data fetching with React Query
    const { data: profile } = useQuery({
        queryKey: ['auth', 'profile', session?.user?.id],
        queryFn: fetchProfile,
    });

    // Return data directly, don't sync with local state
    return {
        user: profile, // Direct data return
        isLoading,
        // ... other values
    };
}
```

### 2. **Use Stable References**

```typescript
// ✅ GOOD - Stable function references
const handleSignOut = useCallback(async () => {
    await signOut();
}, [signOut]);

// ✅ GOOD - Memoized objects
const config = useMemo(
    () => ({
        staleTime: 60 * 1000,
        gcTime: 10 * 60 * 1000,
    }),
    []
);
```

### 3. **Avoid Circular Dependencies**

```typescript
// ❌ WRONG - Circular dependency
// useAuth -> useAuthStore -> useAuth

// ✅ CORRECT - Clear data flow
// auth/callback -> useAuthStore.setUser
// components -> useAuth (read-only)
```

## 🎯 Specific Patterns for Authentication

### 1. **Auth Hook Structure**

```typescript
export function useAuth() {
    // 1. React Query for server state
    const { data: profile } = useQuery({...});

    // 2. Zustand for client state (loading, etc.)
    const { isLoading, setLoading } = useAuthStore();

    // 3. Computed values
    const isAdmin = profile?.is_admin ?? false;

    // 4. Return data directly, not synced state
    return {
        user: profile,        // Direct from React Query
        isLoading,            // From Zustand
        isAdmin,             // Computed
        signIn,              // Stable function
        signOut,             // Stable function
    };
}
```

### 2. **Callback Pattern for State Updates**

```typescript
// ✅ GOOD - Callback pattern in auth callback
export default function AuthCallbackPage() {
    const { setUser } = useAuthStore(); // Direct store access

    const handleAuth = async () => {
        const profile = await fetchProfile();
        setUser(profile); // Direct state update
    };
}
```

## 🚀 Performance Optimization Tips

### 1. **Query Configuration**

```typescript
const { data: session } = useQuery({
    queryKey: ['auth', 'session'],
    queryFn: fetchSession,
    staleTime: 60 * 1000, // 1분 캐시
    gcTime: 10 * 60 * 1000, // 10분 가비지 컬렉션
    refetchOnWindowFocus: false, // 윈도우 포커스 시 재조회 비활성화
    retry: false, // 재시도 비활성화
});
```

### 2. **Conditional Queries**

```typescript
const { data: profile } = useQuery({
    queryKey: ['auth', 'profile', session?.user?.id],
    queryFn: fetchProfile,
    enabled: !!session?.user?.id, // 세션이 있을 때만 실행
    staleTime: 5 * 60 * 1000, // 5분 캐시
});
```

## 🔍 Debugging Checklist

When encountering infinite loops, check:

1. **Dependency Arrays**: Are all dependencies stable references?
2. **State Updates**: Is setState called in useEffect with state dependencies?
3. **Object References**: Are new objects/functions created on every render?
4. **Circular Dependencies**: Are hooks calling each other in a loop?
5. **React Query**: Are query keys changing on every render?
6. **Zustand**: Are store updates triggering component re-renders?

## 📝 Code Review Checklist

- [ ] All useEffect dependencies are stable references
- [ ] No setState calls in useEffect with state dependencies
- [ ] Objects/functions in dependencies are memoized
- [ ] No circular dependencies between hooks
- [ ] React Query keys are stable
- [ ] Zustand selectors are optimized
- [ ] useCallback/useMemo used for expensive operations

## 🎯 Key Takeaways

1. **React Query for server state, Zustand for client state**
2. **Never put setState in useEffect dependency arrays**
3. **Use useCallback/useMemo for stable references**
4. **Avoid circular dependencies between hooks**
5. **Return data directly from React Query, don't sync with local state**
6. **Keep data flow unidirectional and predictable**
   description:
   globs:
   alwaysApply: false

---

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
