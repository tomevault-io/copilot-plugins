## waitingapp-nuxt

> このプロジェクトの一貫性を保つためのコーディング規約です。

# コーディング規約

このプロジェクトの一貫性を保つためのコーディング規約です。

## 全般

- **言語**: TypeScriptを最大限に活用します。
- **フォーマッター**: [Prettier](https://prettier.io/) の使用を推奨します。

### 推奨 Prettier 設定 (`.prettierrc`)

```json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

---

## ファイル命名規則

- **Vue コンポーネント**: `PascalCase.vue` (例: `PageTitle.vue`, `ServiceItem.vue`)
- **Vue ページ**: `PascalCase.vue` (例: `Services.vue`, `Signup.vue`)
- **Composables / Utils / Stores**: `camelCase.ts` (例: `util.ts`, `user.ts`)

---

## Vue コンポーネント

- **SFC (単一ファイルコンポーネント)**
  - 常に `<script setup lang="ts">` を使用します。

- **Props**
  - `defineProps` を使用し、必ずTypeScriptで型を定義します。
  - デフォルト値がある場合は構造分解を利用します。
  ```typescript
  const { title, visible = true } = defineProps<{
    title: string
    /**
     * どのような役割を果たすpropsなのか説明も記載
     * @default true
     */
    visible?: boolean
  }>()
  ```

- **Emits**
  - `defineEmits` を使用し、必ず型を定義します。
  - イベント名はsingle quoteで囲みません。
  - イベント名は`update:changeData`ではなく、`changeData`
  ```typescript
  const emit = defineEmits<{
    update: [number]
  }>()
  ```

- **テンプレート**
  - `v-if` / `v-for` などのディレクティブを持つ要素には、常に `key` 属性を指定します。ただ、できるだけ`index`は使わずに、`object`の`id`などを使います。もし`id`すらなかった場合には`index`を使用します。

---

## TypeScript

- **型定義**
  - `props` や `emit` だけでなく、変数や関数の引数・戻り値にも可能な限り型を付けます。
  - 型の定義には `type` と `interface` のどちらかを使用しますが、このプロジェクトでは `interface` を優先します。
  - `union型`の場合には`type`を使います。
  ```typescript
  interface User {
    id: string;
    name: string;
  }
  ```

- **変数**
  - `const` を基本とし、再代入が必要な場合のみ `let` を使用します。`var` は使用しません。
  - `ref` で定義したリアクティブな変数のサフィックス（例: `...Ref`）は不要です。
    ```typescript
    // Good
    const count = ref(0)

    // Bad
    const countRef = ref(0)
    ```

- **文法**
  - できるだけ1行で書きません。
    ```typescript
    // Good
    if (...) {
      return
    }

    // Bad
    if (...) return
    ```

---
## API通信
- **composables/apis**
  - api通信をする関数は必ず`composables/apis`配下に格納する
  - `useFetch`や`$fetch`を利用する

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hyoilll)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hyoilll)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
