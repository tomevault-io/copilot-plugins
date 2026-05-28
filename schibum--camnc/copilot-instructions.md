## react-error-handling

> For components that use async operations (like WebRTC connections), always use `react-error-boundary` instead of custom class-based error boundaries.


# React Error Handling Patterns

## Error Boundaries with react-error-boundary

For components that use async operations (like WebRTC connections), always use `react-error-boundary` instead of custom class-based error boundaries.

## Hook Rules Compliance

Never violate React's Rules of Hooks by calling hooks conditionally in try-catch blocks:

```tsx
// ❌ Wrong - Violates Rules of Hooks
const MyComponent = () => {
  try {
    const data = useAsyncData();
  } catch (err) {
    useEffect(() => onError(), []); // Conditional hook call!
  }
};

// ✅ Correct - Let ErrorBoundary catch errors
const MyComponent = () => {
  const data = useAsyncData(); // Let this throw, ErrorBoundary will catch
  return <div>{data}</div>;
};
```

---
> Source: [Schibum/camNC](https://github.com/Schibum/camNC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
