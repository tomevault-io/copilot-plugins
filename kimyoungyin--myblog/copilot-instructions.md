## server-client-component-props

> Rules for handling props between Server and Client Components in Next.js


# Server Component vs Client Component Props Rules

## 🚨 Critical Rules (MUST FOLLOW)

### 1. **Never Pass Functions to Client Components from Server Components**

```typescript
// ❌ WRONG - Server Component passing function to Client Component
export default function ServerPage() {
    const handleClick = () => console.log('clicked'); // Function in Server Component

    return (
        <ClientButton onClick={handleClick} /> // This causes serialization error
    );
}

// ✅ CORRECT - Use Link or form actions instead
export default function ServerPage() {
    return (
        <Link href="/new-page">Go to page</Link> // Use Link for navigation
    );
}
```

### 2. **Server Component Props Restrictions**

- **✅ ALLOWED**: Primitive values (string, number, boolean)
- **✅ ALLOWED**: Plain objects (without functions)
- **✅ ALLOWED**: Arrays (without functions)
- **❌ FORBIDDEN**: Functions (event handlers, callbacks)
- **❌ FORBIDDEN**: Complex objects with methods
- **❌ FORBIDDEN**: React refs

### 3. **Client Component Identification**

```typescript
// Client Component - has 'use client' directive
'use client';
export const ClientComponent = ({ onClick }: { onClick: () => void }) => {
    return <button onClick={onClick}>Click me</button>;
};

// Server Component - no 'use client' directive
export default function ServerPage() {
    return <div>Server rendered content</div>;
}
```

## 🔧 Solutions for Common Patterns

### 1. **Navigation Instead of Callbacks**

```typescript
// ❌ WRONG - Function callback
<SortSelector onSortChange={(sort) => handleSort(sort)} />

// ✅ CORRECT - Link with URL params
<SortSelector currentSort={currentSort} />
// Inside SortSelector: <Link href={`?sort=${option.value}`} />
```

### 2. **Form Actions Instead of Event Handlers**

```typescript
// ❌ WRONG - onClick handler
<button onClick={() => handleSubmit(data)}>Submit</button>

// ✅ CORRECT - Form with Server Action
<form action={handleSubmit}>
    <button type="submit">Submit</button>
</form>
```

### 3. **State Management in Client Components**

```typescript
// ❌ WRONG - Server Component managing state
export default function ServerPage() {
    const [count, setCount] = useState(0); // This won't work

    return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// ✅ CORRECT - Client Component for state
'use client';
export const CounterButton = () => {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(count + 1)}>{count}</button>;
};
```

## 🎯 Best Practices

### 1. **Component Architecture**

- **Server Components**: Data fetching, static content, SEO-friendly content
- **Client Components**: Interactivity, state management, event handlers
- **Minimal Client Components**: Keep client components as small as possible

### 2. **Props Design**

```typescript
// ✅ GOOD - Server Component with primitive props
interface ServerComponentProps {
    title: string;
    count: number;
    items: string[];
    metadata: { id: string; name: string };
}

// ❌ BAD - Server Component with function props
interface BadServerComponentProps {
    onAction: () => void; // Function - will cause error
    handleSubmit: (data: any) => void; // Function - will cause error
}
```

### 3. **Error Prevention Checklist**

- [ ] Does the component have `'use client'` directive?
- [ ] Are you passing functions as props from Server Components?
- [ ] Are you using event handlers in Server Components?
- [ ] Are you managing state in Server Components?
- [ ] Are you using browser APIs in Server Components?

## 🚀 Migration Patterns

### 1. **From Function Props to URL Params**

```typescript
// Before: Function callback
<SortSelector onSortChange={handleSort} />

// After: URL params with Link
<SortSelector currentSort={currentSort} />
// SortSelector uses: <Link href={`?sort=${value}`} />
```

### 2. **From Event Handlers to Form Actions**

```typescript
// Before: onClick handler
<button onClick={() => handleAction(data)}>Action</button>

// After: Form with Server Action
<form action={handleAction}>
    <input type="hidden" name="data" value={JSON.stringify(data)} />
    <button type="submit">Action</button>
</form>
```

## 🔍 Debugging Tips

### 1. **Common Error Messages**

```
Error: Event handlers cannot be passed to Client Component props.
Error: Functions cannot be serialized as props from Server Components.
Error: Objects with methods cannot be serialized.
```

### 2. **Debugging Steps**

1. **Identify the component type**: Server vs Client
2. **Check props being passed**: Look for functions or complex objects
3. **Verify component boundaries**: Ensure proper separation
4. **Use React DevTools**: Check component hierarchy and props

### 3. **Quick Fixes**

- **Function props**: Convert to URL params or form actions
- **Event handlers**: Move to Client Components
- **State management**: Use Client Components or Server Actions
- **Complex objects**: Simplify to plain data structures

## 📚 Related Concepts

- **React Server Components**: Server-side rendering without hydration
- **Serialization**: Converting data to transferable format
- **Hydration**: Client-side JavaScript attachment to server HTML
- **Server Actions**: Form-based mutations in Server Components

Remember: **Server Components render on the server and must serialize all props to HTML. Functions and complex objects cannot be serialized.**

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
