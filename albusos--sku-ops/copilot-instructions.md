## frontend-conventions

> Frontend React conventions — TanStack Query, api-client, shared components


# Frontend Conventions

## Data fetching — always use TanStack Query

Never use raw `axios` or manual `useEffect` + `useState` for data fetching. All server state goes through TanStack Query hooks in `src/hooks/`.

```jsx
// ❌ BAD
const [data, setData] = useState([]);
useEffect(() => {
  axios.get('/api/withdrawals').then(r => setData(r.data));
}, []);

// ✅ GOOD
const { data = [] } = useWithdrawals();
```

If no hook exists for an endpoint, create one in the appropriate `src/hooks/use*.js` file using `api-client.js`.

## API calls — always use api-client

All API calls go through `src/lib/api-client.js`. Never import `axios` directly in components or pages.

```jsx
// ❌ BAD — direct axios import in a component
import axios from "axios";
const res = await axios.get(`${API}/chat/status`);

// ✅ GOOD
import api from "@/lib/api-client";
const data = await api.chat.status();
```

## Shared components

Use existing shared components before creating new ones:
- `StatusBadge` / `StockBadge` from `@/components/StatusBadge`
- `Panel` / `SectionHead` from `@/components/Panel`
- `DataTable` from `@/components/DataTable`
- `ViewToolbar` from `@/components/ViewToolbar`

Never define a local copy of a shared component inside a page or modal.

## Page components

Page components orchestrate; they do not implement. Extract sub-components when a page exceeds ~300 lines.

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
