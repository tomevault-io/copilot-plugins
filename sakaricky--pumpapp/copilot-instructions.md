## react-patterns

> React patterns — functional components, arrow + destructuring, hooks


# React patterns

- Use **functional components only** (no class components).
- Use arrow function components with destructured props where applicable.
- Define a **Props type** for each component: `Props` or `ComponentNameProps` interface.

```tsx
// ❌ Avoid
function UserCard(props) {
  return <div>{props.name}</div>;
}

// ✅ Prefer
interface UserCardProps {
  name: string;
  onSave: () => void;
}
const UserCard = ({ name, onSave }: UserCardProps) => (
  <div>{name}</div>
);
```

- Extract reusable logic into **custom hooks** (e.g. `useAuth`, `useProducts`), not duplicated or heavy logic inside components.
- Name components and component files in **PascalCase** when one component per file.
- Custom hooks must start with `use` and follow Rules of Hooks (only at top level, not in conditionals).

---
> Source: [SakaRicky/PumpApp](https://github.com/SakaRicky/PumpApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
