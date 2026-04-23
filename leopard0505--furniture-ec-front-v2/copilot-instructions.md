## furniture-ec-front-v2

> React 命名規則


# 命名規則

- **コンポーネント名**: PascalCase（ファイル名と一致）
- **Props 型名**: `{ComponentName}Props`
- **カスタムフック**: `use{Name}`形式、ファイル名も`use{Name}.tsx`

```tsx
// ✅ 良い例
// Button.tsx
interface ButtonProps {
  text: string;
  onClick?: () => void;
}

export function Button({ text, onClick }: ButtonProps) {
  return <button onClick={onClick}>{text}</button>;
}

// useCart.tsx
export const useCart = () => {
  // ...
};
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Leopard0505) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
