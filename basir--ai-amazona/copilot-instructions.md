## ai-amazona

> - Record fixes for mistakes or corrections to avoid repetition in the `Lessons` section.

# Cursor Rules

## Instructions

- Record fixes for mistakes or corrections to avoid repetition in the `Lessons` section.
- Organize thoughts and plan steps before starting a task in the `Scratchpad` section.
- Clear old tasks if necessary.
- Use todo markers for progress tracking:
  - `[X]` Completed tasks
  - `[ ]` Pending tasks
- Update Scratchpad after completing subtasks.
- Reflect and plan after milestones for better task management.
- Always refer to Scratchpad before planning the next step.

## Lessons

1. Use `npx shadcn@latest add [component]` instead of `npx shadcn-ui@latest add [component]` when installing Shadcn UI components.
2. In Next.js 14+, page props params must be typed as a Promise. Example:
   ```typescript
   type tParams = Promise<{ id: string }>
   interface PageProps {
     params: tParams
   }
   ```
   Then await the params in the component:
   ```typescript
   export default async function Page(props: PageProps) {
     const { id } = await props.params
   }
   ```
3. Use `const session = await auth()` instead of `const session = await getServerSession(authOptions)` for Next.js authentication. The new `auth()` function is the recommended way to get the session in Next.js Auth v5.
4. When importing `useRouter` from 'next/navigation', the component must be marked as a client component using the `'use client'` directive at the top of the file, as this hook only works on the client side.

## Scratchpad

### 1. Project Setup and Configuration [X]

- [x] Initialize Next.js 15 project with TypeScript
- [x] Set up project structure and folders
- [x] Configure ESLint and Prettier
- [x] Install and configure dependencies:
  - Shadcn UI components
  - Lucide icons
  - Zod for validation
  - Zustand for state management
  - Recharts for analytics
  - Resend for emails
  - Uploadthing for file uploads
  - Prisma ORM
  - PostgreSQL database
  - NextAuth.js beta for authentication
  - Stripe for payments

### 2. Database and Authentication [X]

- [x] Set up PostgreSQL database
- [x] Configure Prisma schema:
  - User model
  - Product model
  - Category model
  - Order model
  - Review model
  - Cart model
- [x] Implement NextAuth.js authentication:
  - Email/Password
  - OAuth providers (Google, GitHub)
  - JWT handling
  - Protected routes

### 3. Core Features - Customer Side [ ]

- [x] Home Layout:
  - Create `(home)` folder in `app` directory
  - Header with logo, search bar, and navigation
  - Footer with links and social media
- [x] Homepage:
  - banner carousel
  - latest products
- [x] Products Catalog:
  - Sidebar with categories and filters
  - Search results
  - Product grid
  - Pagination
- [x] Product pages:
  - Create product detail page layout
  - Implement image gallery with thumbnails
  - Add product information section:
    - Title, price, description
    - Stock status
    - Add to cart button
  - Reviews and ratings section:
    - Display existing reviews
    - Add review form for authenticated users
    - Star rating component
  - Related products section:
    - Show products from same category
    - Product card carousel
- [x] Shopping cart:
  - Add/remove items
  - Update quantities
  - Cart persistence
- [x] Checkout process:
  - Shipping information
  - Payment integration (Stripe)
  - Order confirmation
- [x] User dashboard:
  - Order history
  - Profile management
  - Saved addresses
  - Wishlist

### 4. Admin Dashboard [ ]

- [x] Admin authentication and authorization
- [ ] Dashboard overview:
  - [x] Layout and Structure:
    - [x] Create admin dashboard layout with sidebar navigation
    - [x] Implement responsive grid for dashboard widgets
    - [x] Add loading states and error boundaries
  - [x] Key Metrics Cards:
    - [x] Total revenue widget with real data
    - [x] Total orders widget with real data
    - [x] Total customers widget with real data
    - [x] Average order value widget with real data
  - [x] Sales Analytics:
    - [x] Revenue Chart:
      - [x] Implement line chart using Recharts
      - [x] Add daily/weekly/monthly/yearly filters
      - [x] Show revenue trends over time
      - [x] Add tooltip with detailed information
    - [x] Order Statistics:
      - [x] Bar chart for order volume
      - [x] Order status distribution
      - [x] Peak ordering times
  - [x] Recent Orders Table:
    - [x] Implement data table with columns:
      - [x] Order ID
      - [x] Customer name
      - [x] Order total
      - [x] Status
      - [x] Date
    - [x] Add sorting and filtering
    - [x] Quick actions (view, process, update status)
  - [ ] Low Stock Alerts:
    - Products with stock below threshold
    - Quick restock actions
    - Stock level indicators
  - [ ] Top Products:
    - Best-selling products list
    - Revenue by product
    - Stock status
  - [ ] Customer Insights:
    - New vs returning customers
    - Customer acquisition chart
    - Top customers by revenue
  - [ ] Real-time Updates:
    - Implement WebSocket connection
    - Live order notifications
    - Stock level updates
  - [ ] Export and Reports:
    - CSV/PDF export functionality
    - Custom date range selection
    - Report generation
- [ ] Product management:
  - CRUD operations
  - Bulk actions
  - Image upload (Uploadthing)
- [ ] Order management:
  - Order processing
  - Status updates
  - Refund handling
- [ ] User management:
  - Customer list
  - Admin privileges
  - User actions

### 5. Advanced Features [ ]

- [ ] Real-time notifications
- [ ] Email system (Resend):
  - Order confirmations
  - Shipping updates
  - Password reset
- [ ] Search optimization
- [ ] Performance optimization:
  - Image optimization
  - Caching strategies
  - API optimization
- [ ] Analytics and reporting

### 6. Testing and Deployment [ ]

- [ ] Unit testing
- [ ] Integration testing
- [ ] E2E testing
- [ ] Security audit
- [ ] Production deployment:
  - Environment setup
  - CI/CD pipeline
  - Monitoring
  - Backup strategy

### 7. Documentation [ ]

- [ ] API documentation
- [ ] User guide
- [ ] Admin documentation
- [ ] Deployment guide

---
> Source: [basir/ai-amazona](https://github.com/basir/ai-amazona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
