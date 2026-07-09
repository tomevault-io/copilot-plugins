## figma-mcp-experiment-2

> This project demonstrates Claude Code's ability to transpose Figma designs into production-quality React TypeScript components. Each Figma mockup is implemented as a separate route, showcasing component composition, design system adherence, and responsive layouts.

# Figma to React Showcase

## Overview

This project demonstrates Claude Code's ability to transpose Figma designs into production-quality React TypeScript components. Each Figma mockup is implemented as a separate route, showcasing component composition, design system adherence, and responsive layouts.

**Tech Stack:**
- Vite
- React 18+
- TypeScript
- Tailwind CSS
- Shadcn UI
- React Router
- Recharts

**Objective:** Demonstrate agentic coding workflows: observe Figma designs via MCP, decompose into semantic component hierarchies, and implement with best practices.

---

## TypeScript Standards

### Critical Rules
- **NEVER use `any` type** - use `unknown` if type is genuinely uncertain
- Enable `strict: true` in tsconfig.json
- Always define explicit prop interfaces
- Use discriminated unions for variants: `type Status = 'idle' | 'loading' | 'error'`
- Prefer `interface` for object shapes, `type` for unions/intersections

### Component Types
```typescript
// ✅ Correct
import { useState } from 'react'; // Only import what you use (hooks, types)

interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  onClick?: () => void;
}

export const Button: React.FC<ButtonProps> = ({ label, variant = 'primary', onClick }) => {
  // Implementation
};

// ❌ Never do this
export const Button = (props: any) => { /* ... */ };

// Note: React 18+ JSX transform - no need for `import React from 'react'`
```

### Type Organization
- **Component props** → Define inline in component file
- **Domain/data models** → Extract to route-specific `types/` folder (Product, Order, etc.)

```typescript
// routes/shop/types/product.ts - Route-specific domain model
export interface Product {
  id: number;
  title: string;
  price: number;
  image: string;
}

// routes/shop/components/organisms/ProductGrid.tsx - Props inline, import domain model
import type { Product } from '../../types/product';

interface ProductGridProps {  // Inline - component-specific
  products: Product[];
  onProductClick?: (id: number) => void;
}

export const ProductGrid: React.FC<ProductGridProps> = ({ products }) => { };
```

---

## Project Structure

```
src/
├── routes/                 # ALL routes (complete isolation)
│   ├── home/               # Navigation hub route
│   │   └── HomePage.tsx    # Simple page - no components/ needed
│   ├── shop/
│   │   ├── components/
│   │   │   ├── molecules/ # ProductCard, SearchBar, HeaderAuth
│   │   │   └── organisms/ # Header, FilterMenu, ProductGrid, Footer
│   │   ├── types/
│   │   │   └── product.ts # Route-specific types
│   │   └── ShopPage.tsx   # Page component
│   └── dashboard/
│       ├── components/
│       │   ├── molecules/ # StatCard, DonutChart, BarChart
│       │   └── organisms/ # Sidebar, TopNav, RecentOrders
│       ├── types/
│       │   └── order.ts   # Route-specific types
│       └── DashboardPage.tsx
├── components/
│   └── ui/                 # Shadcn ONLY (auto-generated, required)
├── lib/                    # Utilities (utils.ts, cn function)
├── App.tsx                 # Router configuration
└── main.tsx               # Entry point
```

### File Organization Rules
- **Pure route isolation**: Each route is completely self-contained with components, types, and page in `routes/[route-name]/`
- **No shared components**: Import Shadcn components directly from `@/components/ui/` (no wrappers needed)
- **Only create what you need**:
  - Simple pages (like home) → Just the page file, no components/
  - Complex pages → Create components/ and types/ as needed
  - Custom hooks → Create in route-specific hooks/ directory (not top-level)
  - Atoms → Only if route needs custom wrappers (otherwise import Shadcn directly)
- **Route imports**: Use relative imports within route (`./components/molecules/ProductCard`)
- **Shadcn imports**: Use @ alias to import directly (`@/components/ui/button`)
- **Single component export** → Flat file
- **Multiple related components** → Folder with index.ts

```typescript
// Route-specific component: routes/shop/components/molecules/ProductCard.tsx
import type { Product } from '../../types/product'; // Relative import within route
import { Button } from '@/components/ui/button';     // Direct Shadcn import

export const ProductCard: React.FC<{ product: Product }> = ({ product }) => {
  // Implementation
};

// If route needs custom hook, create route-specific hooks/:
// routes/shop/hooks/useProductFilter.ts
export const useProductFilter = () => {
  // Route-specific hook logic
};

// If route needs custom wrapper, create route-specific atoms/:
// routes/shop/components/atoms/ShopButton.tsx
import { Button as ShadcnButton } from '@/components/ui/button';

export const ShopButton: React.FC = (props) => (
  <ShadcnButton className="bg-shop-primary" {...props} />
);
```

### Path Aliases (Required)
Configure in `vite.config.ts` and `tsconfig.json`:
```typescript
// vite.config.ts
import path from 'path';
export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});

// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}

// Usage within routes - relative imports
import { Header } from './components/organisms/Header';
import type { Product } from './types/product';

// Shadcn components - @ alias
import { Button } from '@/components/ui/button';
import { cn } from '@/lib/utils';
```

### Routing Structure
**Every page is a route** - including the navigation hub. All routes live in `routes/[route-name]/` with complete isolation.

**Special routes:**
- `routes/home/HomePage.tsx` - Navigation hub with links to all implemented designs (served at `/`)

**Figma design routes:**
- Each Figma design maps to its own route directory (e.g., `routes/shop/`, `routes/dashboard/`)
- User provides Figma URL + route path (e.g., "Implement [URL] at /shop")

**Workflow:**
1. Initial setup: Create `routes/home/HomePage.tsx` at `/` with navigation links
2. For each new design: User specifies route (e.g., "Implement [URL] at /shop")
3. Create route directory with isolated components/types
4. Create page component, add route to App.tsx, add link to HomePage

**Benefits of complete route isolation:**
- Zero special cases - every page follows the same pattern
- Clear separation between different Figma designs
- No component naming conflicts
- Easy to understand what belongs to which route
- Simple to delete/modify a route without affecting others

---

## Component Architecture

### Atomic Design Principles
- **Atoms**: Primitives (Button, Text, Icon) - no dependencies, wrap Shadcn
- **Molecules**: Simple combinations (SearchBar, Card) - composed of atoms
- **Organisms**: Complex sections (Header, ProductGrid) - composed of molecules/atoms
- **Pages**: Route components - compose organisms only (minimal inline JSX)

### Component Extraction Rule
**Extract every logical Figma section into a named component.** Pages compose organisms and provide mock data.

```typescript
// ✅ Correct - Page has mock data and composes organisms
const MOCK_PRODUCTS = [
  { id: 1, title: 'Product 1', price: 29.99 },
  { id: 2, title: 'Product 2', price: 39.99 },
];

export const HomePage = () => (
  <>
    <HeroSection />
    <ProductGrid products={MOCK_PRODUCTS} />
    <CTASection />
  </>
);

// ❌ Wrong - Inline JSX in page
export const HomePage = () => (
  <>
    <section className="hero">
      <div>...</div> {/* 50 lines */}
    </section>
  </>
);
```

### Component Structure Pattern
```typescript
interface ComponentProps {
  // Props
}

export const Component: React.FC<ComponentProps> = (props) => {
  // 1. Hooks (state, effects, custom hooks)
  // 2. Derived values (useMemo)
  // 3. Event handlers (useCallback)
  // 4. Early returns (loading/error states)
  // 5. Render JSX
};
```

---

## Naming Conventions

### Components and Files
- **Components**: PascalCase (`Button.tsx`, `ProductCard.tsx`)
- **Hooks**: camelCase with `use` prefix (`useDebounce.ts`)
- **Utils**: camelCase (`formatPrice.ts`)
- **Types**: PascalCase, descriptive suffixes (`ButtonProps`, `Product`)

### Props and Variables
- Boolean props: `is*`, `has*`, `should*` (`isLoading`, `hasError`)
- Event handlers: `on*` (props), `handle*` (internal) (`onClick`, `handleClick`)
- Descriptive names - avoid abbreviations

```typescript
// ✅ Good
const filteredProducts = products.filter(predicate);
const isLoading = status === 'loading';

// ❌ Bad
const fp = products.filter(predicate);
const loading = status === 'loading';
```

---

## Styling Guidelines

### Tailwind Utilities
- **Use Tailwind utilities only** - No custom CSS files, no CSS modules, no styled-components
- **Approximate Figma to Tailwind scale** - Round spacing/sizing to nearest Tailwind value (no arbitrary values like `w-[247px]`)
- **Brand colors**: Extract unique Figma colors to `tailwind.config.js`. Use Tailwind defaults for grays/common colors
- **Fonts**: Extract primary font from Figma and add to project (see below)
- Organize classes: layout → spacing → typography → colors → states
- Use responsive prefixes: `md:`, `lg:`, `xl:`

**Mapping guide:**
- Spacing: 12px → `gap-3`, 16px → `p-4`, 24px → `gap-6`, 32px → `p-8`
- Colors: #2c2c2c → `slate-800`, #f5f5f5 → `gray-100` (or extract if brand-specific)
- Radius: 8px → `rounded-lg`, 4px → `rounded`, full → `rounded-full`
- Text: 14px → `text-sm`, 16px → `text-base`

```typescript
// ✅ Correct
<button className="flex items-center px-4 py-2 text-sm bg-primary rounded-lg hover:bg-primary/90" />

// ❌ Wrong - no arbitrary values
<button className="px-[16px] py-[8px] text-[14px] bg-[#2c2c2c] rounded-[8px]" />
```

### Font Setup
Extract font family from Figma metadata and configure:

```html
<!-- index.html - Add Google Fonts link in <head> -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

```javascript
// tailwind.config.js - Update font family
export default {
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
    },
  },
};
```

**Common fonts:** Inter, Roboto, Poppins, Open Sans, Montserrat

### Shadcn Integration
- **Import Shadcn components directly** from `@/components/ui/` in all route components
- Install: `npx shadcn@latest add [component]`
- No wrappers needed - use Shadcn components as-is
- If a route needs custom styling, create route-specific atoms in `routes/[route]/components/atoms/`

```typescript
// Direct import - preferred approach
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';

export const MyComponent = () => (
  <Button variant="outline">Click me</Button>
);

// Route-specific wrapper - only if needed
// routes/shop/components/atoms/ShopButton.tsx
import { Button as ShadcnButton } from '@/components/ui/button';

export const ShopButton: React.FC = (props) => (
  <ShadcnButton className="bg-brand-primary hover:bg-brand-secondary" {...props} />
);
```

### Icon Handling
- **Use Heroicons** - Tailwind's official icon library
- Map Figma icons to Heroicons semantically (magnifying glass → `MagnifyingGlassIcon`)
- Use outline variant by default: `@heroicons/react/24/outline`
- Standard sizing: `w-5 h-5` (20px) or `w-6 h-6` (24px)
- Install: `npm install @heroicons/react`

### Charts and Data Visualization
- **Use Recharts** - React charting library for all chart/graph components
- Install: `npm install recharts`
- Supports: Bar charts, Line charts, Pie/Donut charts, Area charts
- Fully responsive with `ResponsiveContainer`
- Use Tailwind colors for chart fills (e.g., `#6366f1` for indigo-500)

### Images and Avatars
- **Product images**: Use `https://placehold.co/WIDTHxHEIGHT` or `https://picsum.photos/WIDTH/HEIGHT`
- **Avatars**: Use `https://api.dicebear.com/7.x/avataaars/svg?seed=UNIQUE_ID`
- Always include `alt` text for accessibility

---

## UI State & Interactivity Scope

**Implement UI state for interactions, skip business logic.**

### Implement (UI State):
- Tabs/accordion expand/collapse with `useState`
- Mobile menu/drawer open/close
- Modal/dialog show/hide
- Dropdown open/close
- Form input controlled state
- Hover/focus/active visual states

### Skip (Business Logic):
- Actual filtering/searching of data
- Form validation and submission
- API calls or data fetching
- Authentication flows
- Shopping cart operations

```typescript
// ✅ Correct - UI state only
export const FilterMenu = () => {
  const [priceRange, setPriceRange] = useState([0, 100]);
  // Slider works, but doesn't filter products
  return <Slider value={priceRange} onValueChange={setPriceRange} />;
};

// ❌ Wrong - Business logic
const filteredProducts = products.filter(p => p.price >= priceRange[0]);
```

---

## Mock Data Strategy

**Mock data lives in page files (3-6 items). Organisms accept data as props.**

```typescript
// ✅ Correct - Mock data in page, organism accepts props
// routes/shop/ShopPage.tsx
import { Header } from './components/organisms/Header';
import { ProductGrid } from './components/organisms/ProductGrid';
import type { Product } from './types/product';

const MOCK_PRODUCTS: Product[] = [
  { id: 1, title: 'Wireless Headphones', price: 79.99, image: 'https://placehold.co/400x300' },
  { id: 2, title: 'Smart Watch', price: 199.99, image: 'https://placehold.co/400x300' },
  { id: 3, title: 'Laptop Stand', price: 49.99, image: 'https://placehold.co/400x300' },
];

export const ShopPage = () => (
  <>
    <Header />
    <ProductGrid products={MOCK_PRODUCTS} />
  </>
);

// routes/shop/components/organisms/ProductGrid.tsx
import type { Product } from '../../types/product';

interface ProductGridProps {
  products: Product[];
}

export const ProductGrid: React.FC<ProductGridProps> = ({ products }) => (
  <div className="grid grid-cols-3 gap-4">
    {products.map(p => <ProductCard key={p.id} {...p} />)}
  </div>
);

// ❌ Wrong - Mock data in organism
export const ProductGrid = () => {
  const MOCK_PRODUCTS = [...];
  return <div>...</div>;
};
```

**Guidelines:**
- Define mock data constants in page files
- Organisms accept data as props (reusable)
- Images: Use `https://placehold.co/WIDTHxHEIGHT` or `https://picsum.photos/WIDTH/HEIGHT`
- Text: Use Figma's actual text if readable, otherwise placeholder text
- Don't create separate `data/` files
- Don't worry about matching exact Figma item count

---

## Workflow: Figma to React

### Step 1: Analyze Figma Design
Use Figma MCP tools (screenshot + metadata approach):
```typescript
// Get visual reference
mcp__figma__get_screenshot({ fileKey, nodeId })

// Get structure and hierarchy
mcp__figma__get_metadata({ fileKey, nodeId })
```

**Think step by step** - analyze the screenshot and metadata to:
- Identify component hierarchy using layer names and nesting
- Map Figma layers to React components (atoms → molecules → organisms)
- Visually approximate colors/spacing to Tailwind scale
- Identify which Shadcn components to use

### Step 2: Implement Bottom-Up
1. Install required Shadcn components (`npx shadcn@latest add [component]`)
2. Build molecules (compose Shadcn components directly)
3. Build organisms (compose molecules/Shadcn components, accept data as props)
4. Assemble page (define mock data, compose organisms with props)
5. Add route to App.tsx
6. Add link to HomePage (routes/home/HomePage.tsx)

### Step 3: Polish
- Apply responsive patterns (see below)
- Use semantic HTML where natural (`<button>`, `<nav>`, `<header>`)

---

## Responsive Design Strategy

**Figma designs will show desktop only. Infer mobile-first responsive behavior.**

### Standard Patterns to Apply:
- **Horizontal layouts** → Stack vertically on mobile (`flex-col md:flex-row`)
- **Multi-column grids** → Single column on mobile (`grid-cols-1 md:grid-cols-2 lg:grid-cols-3`)
- **Navigation menus** → Hamburger menu on mobile (`hidden md:flex`)
- **Spacing** → Reduce on mobile (`px-4 md:px-8`, `gap-4 md:gap-6`)
- **Typography** → Smaller on mobile (`text-2xl md:text-4xl`)
- **Hide secondary content** → Use `hidden md:block` for non-essential elements

### Breakpoints (Tailwind defaults):
- `md:` → 768px (tablet)
- `lg:` → 1024px (desktop)

```typescript
// ✅ Correct - Responsive header
<header className="flex items-center justify-between px-4 py-4 md:px-8 md:py-6">
  <Logo />
  <nav className="hidden md:flex md:gap-6">
    <a href="#">Home</a>
    <a href="#">Products</a>
  </nav>
  <MobileMenuButton className="md:hidden" />
</header>

// ❌ Wrong - Desktop only (breaks on mobile)
<header className="flex items-center justify-between px-8 py-6">
  <Logo />
  <nav className="flex gap-6">...</nav>
</header>
```

---

## Performance

**Avoid inline function/object creation in JSX** - prevents unnecessary re-renders.

```typescript
// ✅ Correct - Define handlers outside render
const handleClick = () => {
  doSomething();
};

return <Button onClick={handleClick} />;

// ❌ Wrong - Inline function creates new reference every render
return <Button onClick={() => doSomething()} />;

// ❌ Wrong - Inline object
return <Component style={{ padding: 10 }} />;
```

Don't over-optimize with `useMemo`/`useCallback` unless needed.

---

## Code Quality

### Linting
- ESLint with TypeScript rules
- React hooks linter
- Prettier for formatting
- Zero warnings/errors before committing

### Git Commits
Use conventional commits:
```
feat: add product listing page from Figma design
fix: resolve button hover state
refactor: extract header into shared component
```

---

## Implementation Checklist

### Initial Setup
- [ ] Initialize Vite + React + TypeScript
- [ ] Install Tailwind CSS and configure
- [ ] Initialize Shadcn UI (`npx shadcn@latest init`)
- [ ] Install React Router
- [ ] Set up project structure (folders)
- [ ] Configure path aliases (`@/components`, etc.)

### For Each Figma Design
- [ ] Inspect design with Figma MCP tools
- [ ] Identify component hierarchy
- [ ] Install required Shadcn components
- [ ] Implement atoms → molecules → organisms
- [ ] Create page component
- [ ] Add route to router
- [ ] Test responsive behavior

---

**Note:** This is a demonstration project. Focus is on clean component architecture and design system adherence, not functional business logic.

---
> Source: [qolu02/figma_mcp_experiment_2](https://github.com/qolu02/figma_mcp_experiment_2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
