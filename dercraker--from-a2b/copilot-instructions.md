## from-a2b

> EXPLAIN how to create a component

---
description: EXPLAIN how to create a component
globs: *.tsx
---
## Context

* Creating a component must ALWAYS follow this rules.

## Rules

- You always use `export function` without "default".
- You always use an object "props" as the first argument of your component, and add type directly in the object.

## Example

```tsx
type MyComponentProps = { 
  prop1: string; 
  prop2: number;
  prop3: number;
}

export const MyComponent = ({prop1, prop2, prop3}: MyComponentProps) => {
  return <div>{prop1}</div>;
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dercraker)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Dercraker)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
