## suteco

> 基本的なコーディング規約（常に適用）


# 基本的なコーディング規約

## 1. 命名規則
- 変数・関数には `camelCase` を使用する。
- クラス・インターフェース・Reactコンポーネントには `PascalCase` を使用する。
- 定数には `UPPER_SNAKE_CASE` を使用する。

## 2. TypeScriptの適切な利用
- 可能な限り明示的な型定義（TypeやInterface）を行い、`any` 型の使用は避ける。
- オプショナルチェーン（`?.`）とNullish Coalescing（`??`）を活用して安全に値を参照する。

```typescript
// ❌ BAD
const user: any = data;
const name = user && user.name ? user.name : 'Guest';

// ✅ GOOD
const user: User = data;
const name = user?.name ?? 'Guest';
```

## 3. 早期リターン (Early Return) の原則
- 深いネストを避け、ガード句を利用して早めに `return` する。
- 条件分岐がシンプルになり、可読性が向上する。

```typescript
// ❌ BAD
function processUser(user: User | null) {
  if (user) {
    if (user.isActive) {
      // メインの処理
    }
  }
}

// ✅ GOOD
function processUser(user: User | null) {
  if (!user || !user.isActive) return;
  // メインの処理
}
```

## 4. コードの再利用とDRY原則
- 複数の場所で同じロジックが必要な場合は、ヘルパー関数やカスタムフックとして切り出す。
- 1つの関数やコンポーネントが長くなりすぎないよう、適切な粒度で分割する。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kanoi-y)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kanoi-y)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
