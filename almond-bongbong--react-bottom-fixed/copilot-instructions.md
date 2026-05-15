## return-types

> When declaring functions on the top-level of a module,

When declaring functions on the top-level of a module,
declare their return types. This will help future AI
assistants understand the function's purpose.

```ts
const myFunc = (): string => {
  return "hello";
};
```

One exception to this is components which return JSX.
No need to declare the return type of a component,
as it is always JSX.

```tsx
const MyComponent = () => {
  return <div>Hello</div>;
};
```

---
> Source: [almond-bongbong/react-bottom-fixed](https://github.com/almond-bongbong/react-bottom-fixed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
