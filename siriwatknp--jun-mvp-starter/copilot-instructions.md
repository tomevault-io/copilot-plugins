## jun-mvp-starter

> React guideline and coding patterns

## Creating UI state

- Always start with `useState` hook.
- Use primitive values as much as possible, avoid using object.
- If there are better approach to create the state for a specific circumstance, **ask first** and provide the why with detail explanation.
- Avoiding overuse of `useEffect`, try to find alternatives first. For example, moving to `useState` initialization if possible.

## React context

- When passing functions through context, make sure that it's wrapped with `useCallback`.
- When passing object through context, make sure that it's wrapped with `useMemo`.

## Component

Prefer using function directly instead of creating new function inside components. Here are examples that you should **NOT** do:

- simple event handlers

```tsx
function Component({ onClick }) {

  function handleClick() {
    onClick?.()
  }
  
  return <button onClick={handleClick}>...</button>
}
```

- simple render

```tsx
function Component() {
  const renderIcon = () => <svg>...</svg>;
  return <div>{renderIcon()}</div>;
}
```

If the component is a pure component, spread the props to the root element. For example, a LoginButton:

```tsx
function LoginButton({ size, icon, children, ...props }) {
  return (
    <button {...props}>
      {icon} <span>{children}</span>
    </button>
  );
}
```

## TypeScript

- Prefer `interface` over `type` if possible for component props
- For props spread to the root element, include `JSX.intrinsicElement` of the root element too.
- Write the description/purpose above each field of the type. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/siriwatknp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
