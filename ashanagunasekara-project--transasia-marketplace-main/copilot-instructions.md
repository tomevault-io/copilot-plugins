## product-tab-filtering-pattern

> Standard pattern for tab-based product filtering with demoTab


# Product Tab Filtering Pattern

For any tab-driven product section (home blocks, product widgets, etc.), follow this pattern:

- Keep tab options as a `TABS` array with stable `id` values.
- Store selected tab in local state: `activeTab`.
- Compute display items with `useMemo` (do not inline filter in JSX).
- Filter by `product.demoTab?.includes(activeTab)`.
- If there is a dedicated "View All" tab, return the full array in component logic.
- Ensure product data in `data/products/*.ts` includes `demoTab` for every item in the rendered array.
- The first tab id must be present in all product `demoTab` arrays for that section.
- Never add synthetic all-tab ids (such as `view-all` or `all-time`) into `demoTab` data.

## Reference Implementation

```tsx
const [activeTab, setActiveTab] = useState<string>("this-week");

const filteredProducts = useMemo(() => {
  if (activeTab === "view-all") return accessoriesProducts2;
  return accessoriesProducts2.filter((product) =>
    product.demoTab?.includes(activeTab),
  );
}, [activeTab]);
```

## Data Example

```ts
{
  id: 10,
  title: "Pocket Power Pro 5000",
  demoTab: ["this-week", "this-month", "this-year"],
}
```

---
> Source: [ashanagunasekara-project/transasia-marketplace-main](https://github.com/ashanagunasekara-project/transasia-marketplace-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
