## saas

> **Objective:** Ensure generated Zustand stores are well-structured, performant, and used appropriately for managing global client-side state.

**Objective:** Ensure generated Zustand stores are well-structured, performant, and used appropriately for managing global client-side state.

**Core Principles:**

*   **Simplicity:** Keep stores focused and minimal.
*   **Client State Only:** Use Zustand *only* for client-side UI state, configuration state, or short-lived state not suitable for server cache (TanStack Query).
*   **Selectivity:** Optimize component re-renders by selecting only the necessary state slices.

**Rules:**

1.  **Store Definition:**
    *   Define stores using `create` from `zustand`.
    *   MUST use TypeScript to define the store's state shape (`interface` or `type`) and actions.
    *   Separate state and actions clearly within the `create` function.
    *   ```typescript
      interface MyState {
        count: number;
        increment: () => void;
        // Add other state properties and action signatures
      }

      const useMyStore = create<MyState>((set, get) => ({
        count: 0,
        increment: () => set((state) => ({ count: state.count + 1 })),
        // Access current state with get(): get().count
      }));
      ```

2.  **Store Usage:**
    *   Access state in components using the hook: `const count = useMyStore(state => state.count);`
    *   MUST use selectors (`state => state.slice`) to subscribe components only to the specific state pieces they need. This prevents unnecessary re-renders when unrelated parts of the store change.
    *   AVOID selecting the entire state object (`useMyStore()`) unless the component genuinely needs access to multiple, frequently changing parts of the state.
    *   Access actions similarly: `const increment = useMyStore(state => state.increment);`

3.  **Store Organization:**
    *   Create multiple, smaller stores focused on specific domains or features (e.g., `useOwnerFilterStore`, `useUserSettingsStore`, `useNotificationsStore`) rather than one monolithic store.
    *   Place store definitions in a dedicated directory (e.g., `/stores`).

4.  **Middleware:**
    *   Consider using `devtools` middleware during development for easier debugging with Redux DevTools: `create(devtools<MyState>(...))`
    *   Use `persist` middleware if state needs to be persisted across sessions (e.g., in localStorage), but use cautiously for non-sensitive UI preferences only.

5.  **Async Actions:**
    *   Handle asynchronous operations within actions as needed, updating state using `set()` upon completion or failure.
    *   ```typescript
      // Example async action
      fetchData: async () => {
        set({ loading: true });
        try {
          const data = await api.fetchSomeData();
          set({ data, loading: false });
        } catch (error) {
          set({ error, loading: false });
        }
      }
      ```
    *   However, prefer TanStack Query for managing server state, including loading/error states related to API calls. Use Zustand actions primarily for orchestrating client-side state changes around async events if necessary.

6.  **AVOID Duplication:**
    *   DO NOT store server state fetched via TanStack Query within Zustand stores. Use TanStack Query's cache as the source of truth for server data. Zustand is for UI/client state.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ilya2ssss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
