## mefood-next-public

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

- **Development**: `npm run dev` - Start Next.js development server
- **Build**: `npm run build` - Build production application
- **Production**: `npm run start` - Start production server
- **Lint**: `npm run lint` - Run ESLint
- **Check Node Version**: `npm run check-version` - Verify Node.js compatibility
- **Deploy**: `npm run deploy` - Build and deploy with PM2

## Database Commands

- **Generate Prisma Client**: `npx prisma generate`
- **Run Migrations**: `npx prisma migrate dev`
- **Database Studio**: `npx prisma studio`
- **Reset Database**: `npx prisma migrate reset`

## Architecture Overview

This is a Next.js 15 application using the App Router with a restaurant management system (one user = one restaurant).

### Tech Stack
- **Framework**: Next.js 15 with App Router
- **UI**: Material-UI (MUI) with custom theme
- **Database**: MySQL with Prisma ORM
- **Authentication**: NextAuth.js v4 with JWT sessions
- **Styling**: Material-UI + CSS-in-JS

### Database Schema
- **User** model that includes restaurant information (merged User + Restaurant)
- **Account/Session** models for NextAuth.js
- **Table** model for restaurant floor plan and table management
- **Category/MenuItem** models for menu management with selections
- **Selection/SelectionOption** models for menu item customizations
- **CustomerSession** model for guest session tracking
- **Order/OrderItem** models for order management with sub-order support
- **Payment/PaymentItem** models for billing with snapshot data
- **RestaurantSettings** model for restaurant configuration
- All restaurant data is directly associated with the User model

### Authentication Flow
- NextAuth.js handles authentication with credentials (email/password)
- JWT sessions store user ID and restaurant information
- Middleware protects authenticated routes
- Custom sign-in/sign-up pages at `/auth/signin` and `/auth/signup`

### Access Control
- Each user owns and manages exactly one restaurant
- All operations are automatically scoped to the authenticated user's restaurant

### Route Structure
- `/restaurant` - Main restaurant dashboard with table management
  - `/restaurant/menu` - Menu display for customers
  - `/restaurant/table/[tableId]` - Table-specific ordering interface
  - `/restaurant/order-staff-view` - Staff order management view
  - `/restaurant/billing` - Billing and payment management
  - `/restaurant/billing-history` - Payment history
  - `/restaurant/receipt` - Receipt printing
- `/restaurant-admin` - Restaurant administration
  - `/restaurant-admin/details` - Restaurant information management
  - `/restaurant-admin/menu` - Menu item and category management
  - `/restaurant-admin/tables` - Table configuration and floor plan
  - `/restaurant-admin/orders` - Order management and kitchen view
- `/menu/[id]` - Public menu display (customer-facing)
- `/auth/*` - Authentication pages (public)
  - `/auth/signin` - Sign in page
  - `/auth/signup` - Sign up page

### Post-Authentication Redirects
- All authenticated users are redirected to `/restaurant`

### Key Components and Utilities
- `lib/auth.ts` - NextAuth.js configuration
- `middleware.ts` - Route protection
- `app/providers.tsx` - App-wide providers (SessionProvider, ThemeProvider)
- `lib/components/Footer.tsx` - Application footer with developer info and links
- `lib/components/Navbar.tsx` - Main navigation component
- `lib/components/RestaurantNavbar.tsx` - Restaurant-specific navigation
- `lib/components/RestaurantHeader.tsx` - Restaurant header component
- `lib/components/MenuItemImage.tsx` - Menu item image display
- `lib/components/FloorPlan.tsx` - Interactive floor plan for table management
- `lib/components/SelectionsManager.tsx` - Menu item selections/options manager
- `lib/components/DarkTextField.tsx` - Dark theme text field component
- `lib/components/DarkSelect.tsx` - Dark theme select component
- `lib/components/PageTitle.tsx` - Page title component
- `lib/components/Breadcrumb.tsx` - Breadcrumb navigation
- `lib/components/LanguageSwitcher.tsx` - Language switching component
- `lib/components/StructuredData.tsx` - SEO structured data
- `lib/hooks/` - Custom React hooks directory
- `lib/utils/imageUrl.ts` - Image URL utilities

### Environment Variables Required
- `DATABASE_URL` - MySQL connection string
- `NEXTAUTH_SECRET` - JWT encryption secret
- `NEXTAUTH_URL` - Application URL

### Development Patterns
- Use Prisma Client for all database operations
- Implement proper error handling in API routes
- Follow Material-UI theming conventions
- All operations are automatically scoped to the current user

## Deployment

### Prerequisites
- Node.js 18.17.0 or higher (run `npm run check-version` to verify)
- PM2 (optional, for process management): `npm install -g pm2`
- MySQL database configured and accessible

### Deployment Steps

1. **Prepare Environment**
   - Set all required environment variables in `.env.production`
   - Ensure database is accessible and migrations are up to date
   ```bash
   npx prisma migrate deploy
   ```

2. **Deploy Application**
   ```bash
   npm run deploy
   ```
   This command will:
   - Check Node.js version compatibility
   - Build the Next.js application
   - Create necessary directories (public/uploads)
   - Start/reload the application with PM2 (if available)

3. **PM2 Management** (if using PM2)
   - View status: `pm2 list`
   - View logs: `pm2 logs mefood-app`
   - Stop app: `pm2 stop mefood-app`
   - Restart app: `pm2 restart mefood-app`
   - Delete app: `pm2 delete mefood-app`

4. **Manual Deployment** (without PM2)
   ```bash
   npm run build
   npm run start
   ```

### Production Configuration
- Application runs on port 5544 in production
- PM2 configuration in `ecosystem.config.js`
- Logs stored in `./logs/` directory (when using PM2)
- Max memory restart set to 1GB
- Auto-restart enabled for crashes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KarnYong) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
