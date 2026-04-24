## braindrop-mobile

> - **Always use `@ui` alias** for UI components instead of direct imports from `src/core/ui/*`

# Coding Standards

## Import Rules

- **Always use `@ui` alias** for UI components instead of direct imports from `src/core/ui/*`
- **Never import** `Text` or `Button` directly from `react-native` - use `@ui` components instead
- **Use relative imports** for local files within the same feature
- **Use absolute imports** with `@ui` alias for UI components

## TypeScript Standards

- **Strict mode is enabled** - always provide proper types
- **Use interfaces** for object shapes and API responses
- **Use type aliases** for union types and complex types
- **Enable decorators** - the project uses `experimentalDecorators` and `emitDecoratorMetadata`

## React/React Native Standards

- **Use functional components** with hooks
- **Follow React Hooks rules** - hooks must be called at the top level
- **Use proper dependency arrays** in useEffect and other hooks
- **Prefer destructuring** for props and state
- **Use proper key props** for lists and fragments

## Code Style

- **Use semicolons** at the end of statements
- **Use single quotes** for strings
- **Use trailing commas** in objects and arrays
- **Sort JSX props** with shorthand props last
- **Remove unused imports** - the project has `unused-imports` plugin enabled

## Error Handling

- **Use Sentry** for error tracking in production
- **Handle async errors** properly with try/catch
- **Provide fallback UI** for error states

## Performance

- **Use React.memo** for expensive components
- **Optimize re-renders** with proper dependency arrays
- **Use useCallback** and useMemo when appropriate
  description:
  globs:
  alwaysApply: false

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CyrilDesch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
