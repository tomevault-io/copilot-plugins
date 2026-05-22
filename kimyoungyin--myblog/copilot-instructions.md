## react-patterns

> - **ALWAYS** use functional components with hooks

# React Patterns & Functional Programming (MANDATORY)

## ⚛️ **Component Structure (MUST FOLLOW)**

- **ALWAYS** use functional components with hooks
- **NEVER** use class components
- **MUST** use `React.FC<Props>` type annotation for components
- **ALWAYS** destructure props in function parameters

```typescript
// ✅ CORRECT - Functional component with proper typing
export const MarkdownEditor: React.FC<MarkdownEditorProps> = ({
    initialTitle = '',
    initialContent = '',
    action,
    ...props
}) => {
    // Component logic
};

// ❌ WRONG - Class component or improper typing
export class MarkdownEditor extends React.Component { ... }
```

## 🎯 **State Management (CRITICAL)**

- **ALWAYS** use `useState` for local component state
- **ALWAYS** use `useCallback` for functions passed as props
- **ALWAYS** use `useMemo` for expensive calculations
- **NEVER** mutate state directly - always use setter functions

```typescript
// ✅ CORRECT - Proper state management
const [title, setTitle] = useState(initialTitle);
const handleTitleChange = useCallback((value: string) => {
    setTitle(value);
}, []);

// ❌ WRONG - Direct mutation or missing dependencies
const handleTitleChange = (value: string) => {
    title = value; // Direct mutation
};
```

## 🚫 **FORBIDDEN React Practices**

- **NEVER** use class components
- **NEVER** mutate state or props directly
- **NEVER** create functions inside render without useCallback
- **NEVER** use useEffect without proper dependency arrays
- **NEVER** use refs for imperative DOM manipulation unless absolutely necessary

## ✅ **REQUIRED Functional Programming Practices**

- **ALWAYS** use pure functions when possible
- **ALWAYS** prefer `map`, `filter`, `reduce` over loops
- **ALWAYS** use immutable data patterns
- **ALWAYS** handle side effects in useEffect or event handlers only

## 🖥️ **Server/Client Component Patterns (CRITICAL)**

- **ALWAYS** prefer server components by default
- **ONLY** use client components when absolutely necessary (interactivity, browser APIs, state management)
- **ALWAYS** isolate client-side logic to the smallest possible component
- **NEVER** make entire pages client components unless required

### **Server Component Best Practices**

```typescript
// ✅ CORRECT - Server component with async data fetching
export default async function PostPage({ params }: PostPageProps) {
    const post = await getPostAction(postId);
    
    return (
        <div>
            <h1>{post.title}</h1>
            <MarkdownRenderer content={post.content} />
            <ToEditButton postId={postId} /> {/* Client component */}
        </div>
    );
}
```

### **Client Component Isolation**

```typescript
// ✅ CORRECT - Minimal client component for interactive features only
'use client';

export default function ToEditButton({ postId }: { postId: number }) {
    const { user } = useAuthStore(); // Client-side state only
    
    if (!user?.is_admin) return null;
    
    return (
        <Button asChild>
            <Link href={`/admin/posts/${postId}/edit`}>
                <Edit className="h-4 w-4" />
                수정
            </Link>
        </Button>
    );
}
```

### **When to Use Client Components**

- **✅ USE** for interactive elements (buttons, forms, dropdowns)
- **✅ USE** for browser APIs (localStorage, window, document)
- **✅ USE** for state management (useState, useReducer)
- **✅ USE** for event handlers (onClick, onChange)
- **❌ DON'T USE** for static content rendering
- **❌ DON'T USE** for data fetching (use server actions instead)

### **Component Composition Pattern**

```typescript
// ✅ CORRECT - Server component with embedded client components
export default function PostPage() {
    return (
        <div>
            {/* Server-rendered content */}
            <PostContent post={post} />
            
            {/* Client-side interactivity only */}
            <LikeButton postId={post.id} />
            <CommentSection postId={post.id} />
        </div>
    );
}
```

## 🏗️ **HTML Structure & Hydration Prevention (CRITICAL)**

### **1. Block vs Inline Element Rules**

- **NEVER** put block-level elements inside inline elements
- **NEVER** put `<div>`, `<section>`, `<article>` inside `<p>`, `<span>`, `<a>`
- **ALWAYS** ensure proper HTML nesting hierarchy

```typescript
// ✅ CORRECT - Proper HTML structure
<span className="block">
  <div className="relative">
    <Image src={src} alt={alt} />
  </div>
</span>

// ❌ WRONG - Block element inside inline element
<p>
  <div>  {/* This causes hydration error */}
    <Image src={src} alt={alt} />
  </div>
</p>
```

### **2. Markdown Rendering Safety**

- **ALWAYS** override default markdown component behavior
- **ALWAYS** prevent invalid HTML nesting
- **ALWAYS** use custom components for images

```typescript
// ✅ CORRECT - Safe markdown rendering
export const SafeMarkdownRenderer = ({ content }: { content: string }) => {
  return (
    <ReactMarkdown
      components={{
        // Prevent p tag wrapping for images
        img: ({ src, alt }) => (
          <span className="block my-4">
            <Image src={src} alt={alt} width={800} height={600} />
          </span>
        ),
        // Override p tag for image-only content
        p: ({ children, ...props }) => {
          const hasImage = React.Children.toArray(children).some(
            child => React.isValidElement(child) && child.type === 'img'
          );
          return hasImage ? <div {...props}>{children}</div> : <p {...props}>{children}</p>;
        }
      }}
    >
      {content}
    </ReactMarkdown>
  );
};
```

### **3. Hydration Error Prevention**

- **ALWAYS** ensure server and client render identical HTML
- **NEVER** use browser-only APIs in server components
- **ALWAYS** handle dynamic content with proper client boundaries

```typescript
// ✅ CORRECT - Proper client boundary for dynamic content
'use client';

export const DynamicImage = ({ src, alt }: { src: string; alt: string }) => {
  const [isLoaded, setIsLoaded] = useState(false);
  
  return (
    <div className="image-container">
      <Image 
        src={src} 
        alt={alt}
        onLoad={() => setIsLoaded(true)}
      />
      {!isLoaded && <div className="loading-placeholder" />}
    </div>
  );
};

// ❌ WRONG - Browser API in server component
export const ServerComponent = () => {
  const [windowSize, setWindowSize] = useState({}); // Hydration error!
  return <div>{windowSize.width}</div>;
};
```

## 🚀 **Performance Optimization Tips**

### **1. Query Configuration**

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

### **2. Conditional Queries**

```typescript
const { data: profile } = useQuery({
    queryKey: ['auth', 'profile', session?.user?.id],
    queryFn: fetchProfile,
    enabled: !!session?.user?.id, // 세션이 있을 때만 실행
    staleTime: 5 * 60 * 1000, // 5분 캐시
});
```

## 🔍 **Debugging Checklist**

When encountering infinite loops, check:

1. **Dependency Arrays**: Are all dependencies stable references?
2. **State Updates**: Is setState called in useEffect with state dependencies?
3. **Object References**: Are new objects/functions created on every render?
4. **Circular Dependencies**: Are hooks calling each other in a loop?
5. **React Query**: Are query keys changing on every render?
6. **Zustand**: Are store updates triggering component re-renders?
7. **HTML Structure**: Are block elements nested inside inline elements?
8. **Hydration**: Does server render match client render?

## 📝 **Code Review Checklist**

- [ ] All useEffect dependencies are stable references
- [ ] No setState calls in useEffect with state dependencies
- [ ] Objects/functions in dependencies are memoized
- [ ] No circular dependencies between hooks
- [ ] React Query keys are stable
- [ ] Zustand selectors are optimized
- [ ] useCallback/useMemo used for expensive operations
- [ ] No block elements inside inline elements
- [ ] Markdown components properly override defaults
- [ ] Server and client render identical HTML
- [ ] No hydration mismatches in development

## 🎯 **Key Takeaways**

1. **React Query for server state, Zustand for client state**
2. **Never put setState in useEffect dependency arrays**
3. **Use useCallback/useMemo for stable references**
4. **Avoid circular dependencies between hooks**
5. **Return data directly from React Query, don't sync with local state**
6. **Keep data flow unidirectional and predictable**
7. **HTML nesting rules are critical - never put `<div>` inside `<p>`**
8. **Markdown rendering needs custom overrides to prevent invalid HTML**
9. **Server/client consistency is mandatory for hydration**

description: React 패턴과 함수형 프로그래밍, HTML 구조 및 hydration 에러 방지를 위한 필수 규칙
globs: ["**/*.tsx", "**/*.ts", "**/*.jsx", "**/*.js"]
alwaysApply: true

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
