## mega-ulgurji

> Uzbek wholesale e-commerce platform ("MegaHome ulgurji" = MegaHome wholesale). Built with Next.js 16 App Router, Firebase backend, Zustand state management, Tailwind CSS 4. Features ERP-like capabilities: stock management, credit tracking (nasiya), invoicing, profit reports, Excel import/export, and Remotion video marketing.

# MegaHome Ulgurji - E-Commerce Platform

## Project Overview
Uzbek wholesale e-commerce platform ("MegaHome ulgurji" = MegaHome wholesale). Built with Next.js 16 App Router, Firebase backend, Zustand state management, Tailwind CSS 4. Features ERP-like capabilities: stock management, credit tracking (nasiya), invoicing, profit reports, Excel import/export, and Remotion video marketing.

## Tech Stack
- **Framework**: Next.js 16.1.1 (App Router, React 19)
- **Language**: TypeScript 5
- **Styling**: Tailwind CSS 4 + Radix UI + Headless UI + shadcn/ui (new-york style)
- **State**: Zustand 5.0.4 (10 persisted/reactive stores)
- **Backend**: Firebase 11.8.1 (Auth + Firestore + Storage)
- **Server**: Firebase Admin 13.4.0 (API routes)
- **Forms**: React Hook Form + Zod validation
- **Email**: Nodemailer (Gmail SMTP)
- **Video**: Remotion (marketing video production)
- **Icons**: Lucide React + React Icons
- **Animations**: AOS (Animate On Scroll)
- **Notifications**: React Hot Toast
- **Data**: xlsx (Excel/CSV import-export), uuid

## Directory Structure
```
app/
├── (auth)/           # Login & signup pages
├── (client-side)/    # Customer-facing pages (home, product, category, cart, orders)
├── admin/            # Admin panel (dashboard, products, categories, orders, customers,
│                     #   nasiya, kirim, invoices, reports, profile)
├── api/              # API routes (delete-user, sendOrderEmail)
├── layout.tsx        # Root layout (AuthProvider, Toaster, fonts)
└── globals.css       # Tailwind theme + custom CSS variables
components/
├── admin/            # Admin UI (Sidebar, Tables, Forms, Search, BulkPriceUpdate,
│                     #   ImportProducts, DashboardSummary, DailySummary, Notifications)
├── auth/             # LoginForm, SignUpForm, ProtectedRoute
├── client/           # Customer UI (Header, Footer, ProductCard, Cart, Modal, Quantity)
├── contents/         # Page content wrappers
└── ui/               # shadcn/ui primitives (button, avatar, dialog, etc.)
store/                # Zustand stores (10 stores)
├── authStore.ts      # Auth state, user data, role checks
├── useCartStore.ts   # Cart items, quantities, totals (persisted)
├── useCategoryStore.ts # Category CRUD with real-time listeners
├── useProductStore.ts  # Product CRUD with real-time listeners
├── useOrderStore.ts  # Order management with status tracking
├── useDraftStore.ts  # Auto-save drafts (24h expiry)
├── useNasiyaStore.ts # Credit/debt tracking
├── useNotificationStore.ts # Admin notifications
├── useStockReceiptStore.ts # Stock receipt (kirim) management
└── useToggleStore.ts # Sidebar toggle state
remotion/             # Video marketing production
├── Root.tsx          # Composition registry
├── styles.ts         # Brand colors, typography, animation presets
├── index.ts          # Remotion Studio entry point
└── compositions/     # Video templates
    ├── ProductShowcase.tsx   # Product image carousel video (5s)
    ├── PromoVideo.tsx        # Sale/discount promo video (8s)
    └── CategoryHighlight.tsx # Category showcase video (6s)
firebase/config.ts    # Firebase client SDK initialization
lib/
├── types.ts          # TypeScript interfaces (ProductT, CategoryI, Order, UserData, ImageT)
├── utils.ts          # Utility functions (cn() class merging)
├── formatPrice.ts    # Price formatting (UZS)
├── exportExcel.ts    # Excel export utilities
├── importExcel.ts    # Excel import with smart column detection (UZ/EN/RU)
├── orderStatus.ts    # Order status constants
└── notificationSound.ts # Web Audio API notifications
hooks/                # Custom hooks (useWhiteBody, useScrollBodyColor)
providers/            # AuthProvider (Firebase onAuthStateChanged)
```

## Firebase Collections
- **"user"**: `{name, email, uid, role: "admin"|"user", phone, time, date}`
- **"categories"**: `{id, name, description, categoryImgUrl[], storageFileId, subcategory[]}`
- **"products"**: `{id, title, price, costPrice?, productImageUrl[], category, description, quantity, stock?, time, date, storageFileId, subcategory}`
- **"orders"**: `{id, clientName, clientPhone, date, basketItems[], totalPrice, totalQuantity, userUid, status?}`
- **"nasiya"**: Credit/debt tracking records
- **"receipts"**: Stock receipt documents
- **"invoices"**: Generated invoices (schyot-faktura)

## Auth Flow
1. Signup creates Firebase Auth user + Firestore "user" doc (default role: "user")
2. Login authenticates via Firebase → fetches user data → stores in Zustand
3. ProtectedRoute checks `isAuthenticated` and `isAdmin()` for admin routes
4. AuthProvider uses `onAuthStateChanged` listener for session persistence

## Key Patterns
- Real-time Firestore listeners (`onSnapshot`) for products/categories
- Zustand stores with `persist` middleware for cart/auth
- Product images stored in Firebase Storage at `products/{storageFileId}/`
- Category images at `categories/{storageFileId}/`
- Draft auto-save with 24-hour expiry and storage cleanup
- Role-based routing: admin → /admin, user → /
- UI language: Uzbek (interface text), English (code/variables)
- Order statuses: yangi, tasdiqlangan, yigʻilmoqda, yetkazilmoqda, yetkazildi, bekor_qilindi
- Excel import supports UZ/EN/RU column headers with smart detection
- Brand colors: #EEEEEE (gray-100), #5e5e5e (gray-200), #6B6B6B (text), #0e141b (black-text)

## Common Commands
```bash
npm run dev              # Start dev server
npm run build            # Production build
npm run lint             # ESLint check
npm run remotion:studio  # Open Remotion Studio (video editor)
npm run remotion:render  # Render marketing videos
```

## Claude Code Development Toolkit (.claude/plugins/mega-dev/)

### Agents (9)
- **api-developer** — API routes, Firebase Admin SDK, Nodemailer
- **bug-hunter** — Debugging, race conditions, edge cases
- **firebase-expert** — Firestore, Auth, Storage, security rules
- **performance-auditor** — Bundle size, queries, renders, caching
- **ui-builder** — React components, Tailwind, shadcn/ui, responsive
- **seo-marketing** — SEO, metadata, Open Graph, Remotion videos, analytics
- **deployment-ops** — Vercel, CI/CD, monitoring, environment management
- **data-analytics** — Reports, dashboards, KPIs, Excel exports
- **i18n-localization** — Multi-language (UZ/RU/EN), currency, dates

### Commands (10)
- `/setup` — Initialize project for development
- `/add-feature` — Plan and implement new features
- `/fix-issue` — Debug and fix bugs
- `/check-build` — Verify production build
- `/deploy-check` — Pre-deployment checklist
- `/audit-security` — Security vulnerability scan
- `/generate-video` — Create Remotion marketing videos
- `/optimize-seo` — SEO audit and optimization
- `/generate-report` — Business analytics reports
- `/run-tests` — Full test suite execution

### Skills (10)
- **nextjs-patterns** — Route groups, dynamic routes, layouts, metadata
- **firebase-patterns** — Firestore CRUD, Auth, Storage, collections
- **zustand-patterns** — Store creation, persist, selectors, Firebase sync
- **tailwind-patterns** — CSS variables, theme, responsive, brand colors
- **ecommerce-qa** — E-commerce testing patterns and flows
- **remotion-video** — Video production templates, rendering pipeline
- **seo-patterns** — Metadata API, JSON-LD, sitemap, OG images
- **testing-patterns** — Playwright E2E, Vitest, a11y, Lighthouse CI
- **api-security** — Auth tokens, rate limiting, validation, CORS
- **pwa-patterns** — Service workers, offline, push notifications, caching

## Known Issues to Fix
- Email credentials hardcoded in `/app/api/sendOrderEmail/route.ts` → move to env vars
- `next.config.ts` has `unoptimized: true` → enable image optimization
- No pagination on products/orders (will slow down at scale)
- Missing error handling in order email API
- Duplicate `FormattedPrice` in hooks/ and utils/
- No stock validation when adding to cart
- Phone validation hardcoded for +998 (Uzbekistan only)
- No auth token verification on delete-user API route
- No rate limiting on API routes

## Deployment
- **Vercel**: Auto-deploys from `main` branch
- **Firebase**: Project ID `mega-ulgurji-1fccf`
- **GitHub**: https://github.com/baxriddin0317/mega-ulgurji.git

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/baxriddin0317)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/baxriddin0317)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
