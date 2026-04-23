## nextjs-portfolio

> - Storybook は **CSF3 形式**で記述すること

## 基本ルール

- Storybook は **CSF3 形式**で記述すること
- `Meta` と `StoryObj` を使用する
- TypeScript の型安全を維持するため、`Meta<typeof Component>` を定義する
- 各ストーリーには `args` を定義し、`Controls` による操作が可能であること
- `argTypes`には、引数があれば型情報を追加する
- 相互作用テストが必要な場合は `play` 関数を使用する
- Story ファイルの拡張子は `.stories.tsx` とする
- **正常パターンと引数で値が変わる場合（真偽値がある場合）は、複数のストーリーを作成してください**
- **異常パターンのストーリーも1つは作成してください**
- storybook mcpも活用しながらストーリーを作成してください。

### argTypes の例

```tsx
argTypes: {
  text: {
    description: "タイトル",
  },
  id: {
    description: "1",
  },
},
```

- 相互作用テストが必要な場合は `play` 関数を使用する。
- Story ファイルの拡張子は `.stories.tsx` とする。
- **正常パターンと引数で値が変わる場合（真偽値がある場合）は、複数のストーリーを作成してください**
- **異常パターンのストーリーも1つは作成してください**

---

## Storybook 作成手順

### 1. コンポーネントのインポート

```tsx
import type { Meta, StoryObj } from "@storybook/nextjs-vite"
import { ComponentName } from "./ComponentName"
```

### 2. Meta の定義

```tsx
const meta: Meta<typeof ComponentName> = {
  title: "UI/ComponentName",
  component: ComponentName,
  parameters: {
    layout: "centered",
    nextjs: {
      appDirectory: true,
    },
  },
  tags: ["autodocs"],
}

export default meta
```

**注意**: Next.js App Routerを使用している場合は、`parameters.nextjs.appDirectory: true` を設定してください。

### 3. Story の型エイリアス

```tsx
type Story = StoryObj<typeof ComponentName>
```

### 4. 基本ストーリーの作成

```tsx
export const Default: Story = {
  args: {
    prop1: "value",
    prop2: true,
  },
}

// 異常パターンの例（空の配列、長いテキストなど）
export const EmptyTags: Story = {
  args: {
    portfolio_id: 1,
    portfolio_name: "Reactポートフォリオ",
    portfolio_date: "2025-10-06",
    portfolio_tag: [], // 空の配列
    portfolio_topImg: "/images/myphoto.png",
  },
}
```

### 5. インタラクション（任意）

```tsx
import { within, userEvent, expect } from "@storybook/test"

export const Interactive: Story = {
  args: {
    label: "Click me",
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement)
    const button = canvas.getByRole("button", { name: /click me/i })
    await userEvent.click(button)
    expect(button).toHaveTextContent("Clicked!")
  },
}
```

---

## 注意事項

- コンポーネントのすべてのバリエーションをカバーするストーリーを作成する
- インタラクティブなコンポーネントには `play` 関数でテストを追加する
- `args` は Controls で変更可能にし、ドキュメント性を高める

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/takoyan33) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
