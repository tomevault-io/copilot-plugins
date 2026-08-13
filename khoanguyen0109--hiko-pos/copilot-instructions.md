## frontend-typescript

> TypeScript standards for pos-frontend migration and ongoing development


# TypeScript Standards — `pos-frontend`

## No `any`

Never use `any`. The ESLint rule `@typescript-eslint/no-explicit-any` must pass.

- Use `unknown` for values that cannot be determined at author time (e.g. `catch (e: unknown)`, parsed `JSON.parse`). Narrow with type guards before use.
- Use `never` to mark branches that must be unreachable.
- For third-party library gaps, prefer `as SomeConcreteType` with a comment explaining why, or extend the library's types via module augmentation.

## Component Props

Every React component must have an explicit props interface or type alias.

```tsx
// correct
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
  disabled?: boolean;
}

const Button = ({ label, onClick, variant = "primary", disabled = false }: ButtonProps) => { … };

// wrong — implicit props, missing types
const Button = ({ label, onClick }) => { … };
```

- Prefer `interface` for public, extendable shapes; `type` for unions or mapped types.
- Never use `React.FC` — it hides the return type and complicates generics. Annotate the return type explicitly when it is not obvious.
- Remove all `prop-types` imports once a file is converted; they are redundant with TypeScript.

## Domain Model Types

All domain models live in `src/types/`. Import from there; do not inline ad-hoc object shapes in components.

```ts
// src/types/order.ts
export interface OrderItem {
  dish: string;         // Mongoose ObjectId ref
  quantity: number;
  price: number;
  toppings?: string[];
}

export interface Order {
  _id: string;
  orderNumber: number;
  items: OrderItem[];
  orderStatus: "pending" | "preparing" | "ready" | "completed" | "cancelled";
  totalAmount: number;
  paymentMethod?: "cash" | "card" | "online";
  createdAt: string;
  updatedAt: string;
}
```

Required model files in `src/types/`:

| File | Models |
|------|--------|
| `user.ts` | `User`, `UserRole` |
| `store.ts` | `Store`, `StoreMember` |
| `order.ts` | `Order`, `OrderItem`, `OrderStatus`, `PaymentMethod` |
| `table.ts` | `Table`, `TableStatus` |
| `category.ts` | `Category` |
| `dish.ts` | `Dish` |
| `topping.ts` | `Topping` |
| `customer.ts` | `Customer` |
| `member.ts` | `Member` |
| `promotion.ts` | `Promotion`, `DiscountType` |
| `spending.ts` | `SpendingCategory`, `SpendingVendor`, `Spending` |
| `storage.ts` | `StorageItem`, `StorageImport`, `StorageExport`, `Supplier` |
| `schedule.ts` | `Schedule`, `ShiftTemplate`, `ExtraWork`, `Salary` |
| `api.ts` | `ApiResponse<T>`, `PaginatedResponse<T>`, `ApiError` |
| `index.ts` | Re-export all of the above |

## Redux Slices

- Export `RootState` and `AppDispatch` from `store.ts`.
- Type `initialState` explicitly; derive the slice state type from it with `typeof initialState`.
- Use `PayloadAction<T>` for all reducers.
- Typed hooks (`useAppDispatch`, `useAppSelector`) must be used everywhere — never raw `useDispatch`/`useSelector`.

```ts
// src/redux/hooks.ts
import { useDispatch, useSelector } from "react-redux";
import type { RootState, AppDispatch } from "./store";

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector = <T>(selector: (state: RootState) => T) => useSelector(selector);
```

## HTTP Layer (`src/https/`)

- Every API function must have typed parameters and a typed return `Promise<AxiosResponse<ApiResponse<T>>>`.
- Import response models from `src/types/`.

```ts
// correct
export const getOrders = (params: GetOrdersParams): Promise<AxiosResponse<ApiResponse<Order[]>>> =>
  axiosWrapper.get("/api/order", { params });

// wrong — untyped parameters and implicit return
export const getOrders = (params) => axiosWrapper.get("/api/order", { params });
```

## Async Thunks

Provide explicit generic types to `createAsyncThunk`:

```ts
export const fetchOrders = createAsyncThunk<Order[], GetOrdersParams, { rejectValue: string }>(
  "orders/fetchAll",
  async (params, { rejectWithValue }) => { … }
);
```

## Event Handlers

Always type DOM and React synthetic events concretely:

```tsx
// correct
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => { … };

// wrong
const handleChange = (e) => { … };
```

## Utility Functions

- All parameters and return types must be annotated.
- Avoid implicit `any` from untyped third-party libraries; add `@types/…` or create a local `*.d.ts` declaration.

## File Naming & Extensions

| Source type | Extension |
|-------------|-----------|
| React component | `.tsx` |
| Non-component TypeScript | `.ts` |
| Type-only declaration file | `.d.ts` |

Do not mix `.js` / `.jsx` with `.ts` / `.tsx` in the same logical module once migration starts on that module.

## ESLint

The frontend ESLint config must include:

```json
{
  "@typescript-eslint/no-explicit-any": "error",
  "@typescript-eslint/no-unsafe-assignment": "warn",
  "@typescript-eslint/no-unsafe-member-access": "warn",
  "@typescript-eslint/explicit-function-return-type": ["warn", { "allowExpressions": true }]
}
```

Run `npm run lint` from `pos-frontend/` before committing.

---
> Source: [Khoanguyen0109/Hiko-POS](https://github.com/Khoanguyen0109/Hiko-POS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
