## congress-budget-watch-platform

> 遵循社群最佳實踐，高效、安全地使用 Zustand 進行狀態管理。


# 高效使用 Zustand 的五項最佳實踐

本指南提煉自 TkDodo 的文章 "Working with Zustand"，旨在提供清晰、可行的 Zustand 使用準則，以提升應用程式效能與程式碼品質。

## 1. 只導出客製化 Hooks (Selectors)

這是最重要的原則。永遠不要直接導出 `create` 創建的整個 store 實例。應該為 store 中的每個 state 片段建立並導出專屬的 selector hook。

**理由：**
*   **防止意外訂閱整個 store**：如果直接使用 `const { bears } = useBearStore()`，元件會訂閱整個 store，任何狀態（即使是無關的）變更都會導致不必要的重新渲染。
*   **提供更清晰的 API**：元件只需要導入它真正需要的 hook，例如 `useBears()`，而不需要關心 store 的內部結構。

```typescript
// 不佳：直接導出 store
export const useBearStore = create((set) => ({
  bears: 0,
  fish: 0,
  // ...actions
}));

// 良好：只導出 selector hooks
const useBearStore = create<BearState>((set) => ({
  bears: 0,
  fish: 0,
  // ...actions
}));

export const useBears = () => useBearStore((state) => state.bears);
export const useFish = () => useBearStore((state) => state.fish);
```

## 2. 偏好使用原子化的 Selectors

盡量讓每個 selector 只返回一個 state 的最小單位（例如一個數字、字串或布林值）。

**理由：**
*   **避免效能陷阱**：Zustand 預設使用嚴格相等（`===`）來判斷狀態是否變更。如果 selector 返回一個新物件或陣列（例如 `(state) => ({ bears: state.bears, fish: state.fish })`），即使內容完全沒變，它也會被視為「已變更」，導致不必要的渲染。
*   **保持簡單**：雖然可以使用 `shallow` 比較，但建立多個原子化的 selector hook 更簡單、更直觀。如果一個元件需要多個狀態，就使用多個 hooks。

```typescript
// 不佳：返回物件，可能導致不必要的渲染
const { bears, fish } = useBearStore((state) => ({
  bears: state.bears,
  fish: state.fish,
}));

// 良好：使用兩個獨立的原子化 selector
const bears = useBears();
const fish = useFish();
```

## 3. 將 Actions 與 State 分離

在 store 內部，將更新狀態的函式（actions）與狀態本身（state）分開存放。

**理由：**
*   **效能優化**：Actions 本身是靜態的，它們的引用永遠不會改變。將它們組合在一個物件中，可以建立一個單獨的 hook（例如 `useBearActions`）來獲取所有 actions，而這個 hook 永遠不會觸發元件的重新渲染。
*   **組織清晰**：將狀態和操作狀態的方法分開，使 store 的結構更清晰。

```typescript
const useBearStore = create<BearState>((set) => ({
  bears: 0,
  fish: 0,
  // ⬇️ 將 actions 放在獨立的 "namespace"
  actions: {
    increasePopulation: (by) =>
      set((state) => ({ bears: state.bears + by })),
    eatFish: () => set((state) => ({ fish: state.fish - 1 })),
  },
}));

export const useBears = () => useBearStore((state) => state.bears);

// 🎉 一個 selector 獲取所有 actions，且不會觸發重渲染
export const useBearActions = () => useBearStore((state) => state.actions);

// 在元件中使用
const { increasePopulation } = useBearActions();
```

## 4. 將 Actions 建模為事件，而非 Setters

Actions 應該代表「使用者意圖」或「應用程式事件」（例如 `increasePopulation`），而不是單純的 `setBears`。

**理由：**
*   **業務邏輯內聚**：將業務邏輯保留在 store 內部，而不是散落在各個元件中。元件只負責觸發事件，store 決定如何響應這個事件來更新狀態。
*   **可維護性**：當業務邏輯需要變更時，只需要修改 store 中的 action，而不需要去尋找並修改所有使用到它的元件。

## 5. 保持 Store 的範疇小而專注

與 Redux 提倡的單一全局 store 不同，Zustand 鼓勵你建立多個、小而專注的 stores，每個 store 只負責一塊特定的業務領域或功能。

**理由：**
*   **高內聚，低耦合**：每個 store 都更小、更易於理解和測試。
*   **靈活組合**：如果需要在不同 stores 之間共享狀態，可以使用客製化 hook 來組合它們，而不是建立複雜的 store 結構。

```typescript
// 透過客製化 hook 組合不同的 stores
const currentUser = useCredentialsStore((state) => state.currentUser);
const user = useUsersStore((state) => state.users[currentUser]);
```

參考資料：[Working with Zustand by TkDodo](https://tkdodo.eu/blog/working-with-zustand)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/readr-media) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
