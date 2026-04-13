## tanminh-mobile

> E-commerce platform specializing in tablets (iPad, Samsung Tab) and trade-in services. Vietnamese market, VND currency.

# CLAUDE.md - TanMinh Mobile E-Commerce

## Project Overview

E-commerce platform specializing in tablets (iPad, Samsung Tab) and trade-in services. Vietnamese market, VND currency.

**Live URL:** `http://localhost:3002` (dev) | Domain TBD (production)

## Tech Stack

- **Framework:** Next.js 16 (App Router, Turbopack) + React 19
- **Language:** TypeScript (strict mode)
- **Database:** PostgreSQL + Prisma 7 (driver adapter pattern)
- **Auth:** NextAuth v5 (credentials provider, JWT strategy)
- **Styling:** TailwindCSS v4 (CSS-first config in `src/app/globals.css`)
- **State:** Zustand with persist middleware (localStorage)
- **Validation:** Zod v4 (`zod/v4` import path)
- **UI Libraries:** Recharts, Embla Carousel, TipTap, @tanstack/react-table, Lucide icons, Sonner toasts

## Commands

```bash
pnpm dev              # Dev server (Turbopack)
pnpm build            # Production build
pnpm start -p 3002    # Production server (port 3000 may be occupied)
pnpm db:generate      # Regenerate Prisma client
pnpm db:migrate       # Run migrations
pnpm db:seed          # Seed sample data
pnpm db:studio        # Open Prisma Studio GUI
npx tsc --noEmit      # Type check without building
```

## Architecture

### Directory Structure

```
src/
  app/
    (auth)/          # Login, Register, Forgot/Reset password
    (storefront)/    # Customer-facing pages (Vietnamese URLs)
    admin/           # Admin dashboard (protected by middleware)
    api/             # REST API routes (~44 endpoints)
  components/
    ui/              # 18 design system primitives (Button, Input, Modal, DataTable...)
    layout/          # Header, Footer, MegaMenu, CartDrawer, SearchBar
    home/            # Homepage sections (HeroCarousel, FeaturedProducts...)
    product/         # ProductCard, ProductDetail, ProductFilters
    admin/           # Dashboard widgets (StatsCard, RevenueChart, ProductForm...)
    blog/            # BlogPostDetail
    providers/       # Providers.tsx (SessionProvider + Toaster)
  lib/
    services/        # 13 business logic services (product, order, cart, etc.)
    validations/     # 10 Zod schemas per domain
    hooks/           # useDebounce, useClickOutside, useFetch
    auth.ts          # NextAuth config (credentials + JWT + role in token)
    auth.config.ts   # Auth edge config for middleware
    prisma.ts        # Prisma singleton with pg driver adapter
    upload.ts        # Image upload helper (Vercel Blob)
    utils.ts         # cn(), formatPrice(), formatDateTime(), slugify(), generateOrderNumber()
    constants.ts     # Vietnamese UI labels (ORDER_STATUS_LABELS, PAYMENT_*, TRADE_IN_*)
  stores/
    cart.store.ts    # Zustand cart (client-side, persisted to localStorage)
    ui.store.ts      # UI state (mobile menu, cart drawer, search)
  types/
    index.ts         # Shared TypeScript types
    next-auth.d.ts   # NextAuth session type extension (adds id, role)
```

### Database (27 Prisma Models)

Schema at `prisma/schema.prisma`. Key domains:

| Domain | Models |
|--------|--------|
| Auth | User, Account, Session, VerificationToken, Address |
| Catalog | Product, ProductImage, ProductVariant, Category, ProductCategory, RelatedProduct |
| Cart | Cart, CartItem |
| Orders | Order, OrderItem, OrderStatusHistory |
| Coupons | Coupon, CouponUsage |
| Reviews | Review |
| Wishlist | WishlistItem |
| Trade-in | TradeInRequest, TradeInStatusHistory |
| CMS | BlogCategory, BlogPost, Page, Banner, Setting |

**Key conventions:**
- Prices: `Decimal(12,0)` — integer VND, no floating point
- Order snapshots shipping info (shippingName, shippingPhone, shippingStreet, etc.)
- Category: self-referential `parentId` for unlimited nesting
- Cart: dual architecture — Zustand (client) + Prisma Cart/CartItem (server), synced at checkout
- Guest cart via `cart_session` cookie, merges into user cart on login

### Enums (from Prisma)

- `UserRole`: CUSTOMER, ADMIN, SUPER_ADMIN
- `ProductCondition`: NEW, LIKE_NEW, CPO, USED
- `ProductStatus`: ACTIVE, DRAFT, ARCHIVED
- `OrderStatus`: PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED, REFUNDED
- `PaymentStatus`: PENDING, PAID, FAILED, REFUNDED
- `PaymentMethod`: COD, BANK_TRANSFER, VNPAY, MOMO
- `TradeInStatus`: SUBMITTED, REVIEWING, QUOTED, ACCEPTED, REJECTED, COMPLETED, CANCELLED
- `ReviewStatus`: PENDING, APPROVED, REJECTED

## Code Patterns

### API Routes

All API routes follow this pattern:
```typescript
// src/app/api/[resource]/route.ts
import { NextRequest, NextResponse } from "next/server";
import { someService } from "@/lib/services/some.service";
import { auth } from "@/lib/auth";
import { z } from "zod/v4";

export async function GET(req: NextRequest) {
  try {
    const session = await auth();
    // Auth check if needed
    // Parse query params
    const result = await someService.list(...);
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error instanceof Error ? error.message : "Loi server" },
      { status: 400 }
    );
  }
}
```

**Dynamic route params** are `Promise<{ id: string }>` (Next.js 16):
```typescript
type Params = { params: Promise<{ id: string }> };
export async function GET(_req: NextRequest, { params }: Params) {
  const { id } = await params;
  // ...
}
```

### Service Layer

Services are plain objects with async methods, using Prisma directly:
```typescript
// src/lib/services/product.service.ts
export const productService = {
  async getBySlug(slug: string) { ... },
  async list(page, limit, filters) { ... },
  async create(data: CreateProductInput) { ... },
};
```

### Auth Protection

- **Middleware** (`src/middleware.ts`): Redirects unauthenticated users from `/admin/*` and `/tai-khoan/*`
- **Defense-in-depth**: API routes also check `await auth()` + role
- **Admin check pattern**: `if (role !== "ADMIN" && role !== "SUPER_ADMIN")`

### Input Validation

All inputs validated with Zod v4 before hitting the service layer:
```typescript
import { z } from "zod/v4";
const schema = z.object({ ... });
const data = schema.parse(await req.json());
```

### Client-Side Cart

Zustand store with `persist` middleware (localStorage key: `"cart-storage"`):
- Cart lives client-side for instant UX
- At checkout, cart items sent in request body to `/api/checkout`
- On login, client cart can be synced via `/api/cart/sync`

## Important Gotchas

1. **Zod import**: Always use `from "zod/v4"` (NOT `from "zod"`)
2. **Prisma Decimal**: Prices come as `Decimal` objects from Prisma. Use `Number()` to convert for JSON
3. **Order fields**: Orders use `shippingName`, `shippingPhone`, `shippingStreet` (NOT `customerName`, `address`)
4. **Trade-in fields**: Use `deviceModel` (not `deviceName`), `finalPrice` (not `offeredPrice`), `adminNote` (not `note`), status starts at `SUBMITTED` (not `PENDING`)
5. **Category/Banner ordering**: Field is `position` (NOT `sortOrder`)
6. **Order status**: Use `SHIPPED` (NOT `SHIPPING`)
7. **Port conflict**: Port 3000 may be occupied. Use `pnpm start -p 3002`
8. **AUTH_TRUST_HOST**: Set `AUTH_TRUST_HOST=true` in `.env` when running on non-standard port
9. **Zustand SSR**: `location is not defined` warning during build is benign (persist middleware checks localStorage)
10. **Next.js 16 middleware**: Shows deprecation warning about "proxy" convention — works fine, ignore for now

## Test Accounts (after seed)

| Role | Email | Password |
|------|-------|----------|
| Super Admin | `minhlevu422003@gmail.com` | `admin123` |
| Customer | `customer@test.com` | `customer123` |

## Seed Data

- 8 tablet products (iPad Pro, iPad Air, iPad Gen 10, iPad Mini, Samsung Tab) with variants
- 2 top-level categories (Máy tính bảng, Thu cũ đổi mới) + 4 subcategories (iPad, iPad Pro, iPad Air, Samsung Tab)
- 4 banners, blog posts, 4 static pages, coupons, settings

## Environment Variables

```env
DATABASE_URL="postgresql://..."     # Required
NEXTAUTH_URL="http://localhost:3002" # Match your running port
NEXTAUTH_SECRET="..."               # Change in production!
AUTH_TRUST_HOST=true                 # Required for non-standard ports
BLOB_READ_WRITE_TOKEN=""             # Vercel Blob (auto-set in Vercel dashboard)
NEXT_PUBLIC_APP_NAME="TanMinh Mobile"
NEXT_PUBLIC_APP_URL="http://localhost:3002"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DinhVanMy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DinhVanMy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
