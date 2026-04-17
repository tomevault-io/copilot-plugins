## react-spa-starter

> **This project uses React Router v7 in DATA MODE for SPA development, NOT Framework Mode.**

# React Router v7 Data Mode (SPA) - Cursor Rules

## 🚨 CRITICAL: Data Mode vs Framework Mode

**This project uses React Router v7 in DATA MODE for SPA development, NOT Framework Mode.**

- ✅ **Data Mode**: Uses `createBrowserRouter` with manual route configuration  
- ❌ **Framework Mode**: Uses `app/routes.ts` with auto-generated types

## Critical Package Guidelines

### ✅ CORRECT Packages for Data Mode:
- `react-router` - Main package for routing components, hooks, and router creation

### ❌ NEVER Use in Data Mode:
- `@react-router/dev` - Only for Framework Mode
- `@react-router/node` - Only for server-side Framework Mode  
- `@react-router/serve` - Only for Framework Mode
- `react-router-dom` - Legacy package, use `react-router` instead
- `@remix-run/*` - Old packages, replaced by `@react-router/*`

## Essential Data Mode Architecture

### Router Configuration Pattern
```tsx
import { createBrowserRouter, Navigate } from "react-router";
import { Root } from "./root";
import { DashboardPage } from "./dashboard/dashboard.page";

export const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    children: [
      {
        index: true,
        element: <DashboardPage />,
        loader: dashboardLoader,
        handle: {
          crumb: <DashboardBreadcrumb />,
        },
      },
      {
        path: "products/:id",
        element: <ProductPage />,
        loader: productLoader,
        action: productAction,
      },
      {
        path: "products",
        element: <ProductLayout />,
        children: [
          {
            index: true,
            element: <Navigate to="list" replace />,
          },
          {
            path: "list",
            element: <ProductList />,
            loader: productListLoader,
          },
        ],
      },
    ],
  },
  {
    path: "*",
    element: <NotFound />,
  },
]);
```

### Route Module Pattern (Data Mode)
```tsx
import { LoaderFunction, ActionFunction, redirect } from "react-router";
import { useLoaderData } from "react-router";

// Type-safe loader
export const productLoader: LoaderFunction = async ({ params }) => {
  const product = await getProduct(params.id!);
  if (!product) {
    throw new Response("Product Not Found", { status: 404 });
  }
  return { product };
};

// Type-safe action
export const productAction: ActionFunction = async ({ request, params }) => {
  const formData = await request.formData();
  await updateProduct(params.id!, formData);
  return redirect(`/products/${params.id}`);
};

// Component with typed loader data
interface ProductLoaderData {
  product: Product;
}

export default function ProductPage() {
  const { product } = useLoaderData() as ProductLoaderData;
  
  return (
    <div>
      <h1>{product.name}</h1>
      <Form method="post">
        <input name="name" defaultValue={product.name} />
        <button type="submit">Update</button>
      </Form>
    </div>
  );
}
```

### Layout Routes with Outlet
```tsx
import { Outlet, useNavigation } from "react-router";

export default function ProductLayout() {
  const navigation = useNavigation();
  
  return (
    <div className="layout">
      <nav>
        {/* Sidebar or navigation */}
      </nav>
      <main>
        {navigation.state === "loading" && <LoadingSpinner />}
        <Outlet /> {/* ✅ This renders the matching child route */}
      </main>
    </div>
  );
}
```

## Type Safety in Data Mode

### ✅ Manual Type Definitions:
In Data Mode, you define your own types for loader data and form actions:

```tsx
import { LoaderFunction, ActionFunction } from "react-router";

// Define your loader data types
interface DashboardLoaderData {
  user: User;
  stats: DashboardStats;
}

interface ProductLoaderData {
  product: Product;
  reviews: Review[];
}

// Type-safe loaders
export const dashboardLoader: LoaderFunction = async ({ request }) => {
  const user = await getCurrentUser(request);
  const stats = await getDashboardStats(user.id);
  return { user, stats } satisfies DashboardLoaderData;
};

// Type-safe component usage
export default function Dashboard() {
  const { user, stats } = useLoaderData() as DashboardLoaderData;
  
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <StatsDisplay stats={stats} />
    </div>
  );
}
```

### ✅ Form Action Types:
```tsx
interface ProductFormData {
  name: string;
  price: number;
  description: string;
}

export const productAction: ActionFunction = async ({ request, params }) => {
  const formData = await request.formData();
  
  const productData: ProductFormData = {
    name: formData.get("name") as string,
    price: Number(formData.get("price")),
    description: formData.get("description") as string,
  };
  
  await updateProduct(params.id!, productData);
  return redirect(`/products/${params.id}`);
};
```

## Critical Imports & Patterns

### ✅ Correct Imports for Data Mode:
```tsx
import { 
  createBrowserRouter, 
  RouterProvider,
  Link, 
  NavLink, 
  Form, 
  useLoaderData, 
  useActionData,
  useFetcher, 
  useNavigation,
  useNavigate,
  Outlet,
  Navigate,
  redirect,
  LoaderFunction,
  ActionFunction,
  type LoaderFunctionArgs,
  type ActionFunctionArgs,
} from "react-router";
```

### ❌ DON'T Import Framework Mode Features:
```tsx
// ❌ These don't exist in Data Mode:
// import { index, route, type RouteConfig } from "@react-router/dev/routes";
// import type { Route } from "./+types/product"; // No auto-generated types
// import { href } from "react-router"; // href() doesn't exist in Data Mode
```

## Data Loading & Actions

### Client-Side Data Loading:
```tsx
// All loaders in Data Mode are client-side by default
export const productLoader: LoaderFunction = async ({ params }) => {
  // This runs on the client during navigation
  const response = await fetch(`/api/products/${params.id}`);
  if (!response.ok) {
    throw new Response("Product not found", { status: 404 });
  }
  return { product: await response.json() };
};
```

### Form Handling & Actions:
```tsx
export const productAction: ActionFunction = async ({ request }) => {
  const formData = await request.formData();
  const intent = formData.get("intent");
  
  switch (intent) {
    case "update":
      await updateProduct(formData);
      break;
    case "delete":
      await deleteProduct(formData.get("id") as string);
      return redirect("/products");
    default:
      throw new Response("Invalid intent", { status: 400 });
  }
  
  return { success: true };
};

// In component
<Form method="post">
  <input name="intent" value="update" type="hidden" />
  <input name="name" placeholder="Product name" />
  <button type="submit">Update Product</button>
</Form>
```

## Navigation & Links

### Basic Navigation:
```tsx
import { Link, NavLink, useNavigate } from "react-router";

// Simple links - use string paths directly
<Link to="/products">Products</Link>
<Link to={`/products/${product.id}`}>View Product</Link>

// Active state styling
<NavLink 
  to="/dashboard" 
  className={({ isActive }) => isActive ? "active" : ""}
>
  Dashboard
</NavLink>

// Programmatic navigation
const navigate = useNavigate();
navigate(`/products/${productId}`);
navigate("/products", { replace: true });
```

### Advanced Navigation with Fetchers:
```tsx
import { useFetcher } from "react-router";

function AddToCartButton({ productId }: { productId: string }) {
  const fetcher = useFetcher();
  
  return (
    <fetcher.Form method="post" action="/api/cart">
      <input type="hidden" name="productId" value={productId} />
      <button type="submit">
        {fetcher.state === "submitting" ? "Adding..." : "Add to Cart"}
      </button>
    </fetcher.Form>
  );
}
```

## File Organization & Naming

### ✅ Flexible File Organization:
Data Mode gives you complete freedom in file organization:

```
src/
├── routes/
│   ├── routes.tsx              # Router configuration
│   ├── root.tsx               # Root layout
│   ├── dashboard/
│   │   ├── dashboard.page.tsx # Route component
│   │   ├── loader.ts          # Loader function
│   │   └── breadcrumb.tsx     # Breadcrumb component
│   └── products/
│       ├── product.page.tsx
│       ├── product-list.page.tsx
│       └── actions.ts         # Action functions
```

### File Naming Best Practices:
- Use **descriptive names** that clearly indicate purpose
- Use **kebab-case** for consistency (`product-details.tsx`)
- Organize by **feature** rather than routing conventions
- Separate loaders/actions into their own files when they get large

## Error Handling & Boundaries

### Route Error Boundaries:
```tsx
// In your route configuration
{
  path: "products/:id",
  element: <ProductPage />,
  loader: productLoader,
  errorElement: <ProductErrorBoundary />,
}

// Error boundary component
function ProductErrorBoundary() {
  const error = useRouteError();
  
  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>{error.status} {error.statusText}</h1>
        <p>{error.data}</p>
      </div>
    );
  }
  
  return (
    <div>
      <h1>Oops!</h1>
      <p>{error?.message || "An unexpected error occurred"}</p>
    </div>
  );
}
```

### Throwing Errors from Loaders/Actions:
```tsx
export const productLoader: LoaderFunction = async ({ params }) => {
  const product = await db.getProduct(params.id!);
  if (!product) {
    throw new Response("Product Not Found", { status: 404 });
  }
  return { product };
};
```

## Advanced Patterns

### Pending UI & Navigation State:
```tsx
import { useNavigation, useFetcher } from "react-router";

// Global pending state
function GlobalSpinner() {
  const navigation = useNavigation();
  return navigation.state === "loading" ? <Spinner /> : null;
}

// Optimistic UI with fetchers
function CartItem({ item }: { item: CartItem }) {
  const fetcher = useFetcher();
  
  // Optimistic quantity from form data
  const quantity = fetcher.formData 
    ? Number(fetcher.formData.get("quantity"))
    : item.quantity;
    
  return (
    <fetcher.Form method="post" action="/cart/update">
      <input type="hidden" name="itemId" value={item.id} />
      <input 
        type="number" 
        name="quantity" 
        value={quantity}
        onChange={(e) => fetcher.submit(e.currentTarget.form)}
      />
      <span>{item.product.name}</span>
    </fetcher.Form>
  );
}
```

### Breadcrumbs with Handle:
```tsx
// In router configuration
{
  path: "products/:id",
  element: <ProductPage />,
  loader: productLoader,
  handle: {
    crumb: (data) => <ProductBreadcrumb product={data.product} />,
  },
}

// Breadcrumb rendering
function Breadcrumbs() {
  const matches = useMatches();
  
  const crumbs = matches
    .filter((match) => Boolean(match.handle?.crumb))
    .map((match) => match.handle.crumb(match.data));
    
  return (
    <ol>
      {crumbs.map((crumb, index) => (
        <li key={index}>{crumb}</li>
      ))}
    </ol>
  );
}
```

## Anti-Patterns to Avoid

### ❌ Framework Mode Patterns:
```tsx
// ❌ DON'T use Framework Mode features in Data Mode
import { href } from "react-router"; // Doesn't exist
import type { Route } from "./+types/product"; // No auto-generated types
import { index, route } from "@react-router/dev/routes"; // Not available
```

### ❌ Manual Data Fetching in Components:
```tsx
// ❌ DON'T fetch in components - use loaders instead
function Product() {
  const [data, setData] = useState(null);
  useEffect(() => { 
    fetch('/api/products').then(/* ... */); 
  }, []);
  // Use loader instead!
}
```

### ❌ Manual Form Handling:
```tsx
// ❌ DON'T handle forms manually - use actions instead
const handleSubmit = (e) => {
  e.preventDefault();
  fetch('/api/products', { method: 'POST' });
};
// Use Form component and action instead!
```

### ❌ React Router v6 Patterns:
```tsx
// ❌ DON'T use component routing
<Routes>
  <Route path="/" element={<Home />} />
</Routes>
```

## Essential Data Mode Rules

1. **ALWAYS** use `createBrowserRouter` for router configuration
2. **DEFINE** your own TypeScript interfaces for loader data
3. **USE** `LoaderFunction` and `ActionFunction` types for type safety
4. **CAST** `useLoaderData()` return to your defined interface types
5. **ORGANIZE** files by feature, not by routing conventions
6. **SEPARATE** loaders and actions into their own files when they get large
7. **USE** string paths directly in `Link` and `navigate()` - no `href()` helper

## AI Assistant Guidelines

When working with React Router v7 Data Mode:
- **NEVER suggest Framework Mode features** like auto-generated types or `href()`
- **ALWAYS use `createBrowserRouter`** for route configuration
- **SUGGEST manual type definitions** for loader data and actions
- **RECOMMEND separating loaders/actions** into their own files for large routes
- **USE standard React Router hooks** like `useLoaderData`, `useActionData`, etc.
- **CAST loader data** to TypeScript interfaces for type safety

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mailok) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
