## el-shadai

> **El Shadai** is a comprehensive web-based management and delivery system, designed for a local retail business (likely a pet store/general store). It handles product management, user administration (Customers, Staff, Delivery), order processing, and inventory control.

# El Shadai - Project Context

## Project Overview

**El Shadai** is a comprehensive web-based management and delivery system, designed for a local retail business (likely a pet store/general store). It handles product management, user administration (Customers, Staff, Delivery), order processing, and inventory control.

The application is built as a **Progressive Web App (likely)** or responsive web application using the **Next.js App Router**.

## Tech Stack

*   **Framework:** [Next.js 15+](https://nextjs.org/) (App Router)
*   **Language:** TypeScript
*   **Styling:** SCSS Modules (`*.module.scss`)
*   **Database:** PostgreSQL
*   **ORM:** [Prisma](https://www.prisma.io/)
*   **Authentication:** [NextAuth.js v5](https://authjs.dev/) (Beta)
*   **Internationalization:** [next-intl](https://next-intl-docs.vercel.app/)
*   **Image Hosting:** Cloudinary

## Architecture & Key Directories

The project follows a feature-based structure within the Next.js App Router:

*   **`src/app/`**: Main application routes.
    *   **`(main)/`**: Root layout group.
        *   **`(admin)/`**: Administrative interfaces (Dashboard, Product management).
        *   **`(costumer)/`**: Customer-facing interfaces (Cart, Checkout, Orders, Products).
        *   **`(delivery)/`**: Delivery personnel interfaces.
    *   **`api/`**: Backend API routes (`v1/` for entities like products, users, orders; `auth/` for authentication).
    *   **`auth/`**: Authentication pages (Login, Register).
*   **`src/components/`**: Reusable UI components, categorized by feature (`admin`, `cart`, `shared`, etc.).
*   **`src/lib/`**: Utility functions, hooks, and configuration (Cloudinary, formatters).
*   **`src/prisma/`**: Prisma schema (`schema.prisma`) and configuration. Note: `seed.ts` is in the root `prisma/` folder.
*   **`src/services/`**: Business logic layer interacting with the database/API (e.g., `ProductsService.ts`, `OrderService.ts`).
*   **`src/styles/`**: Global styles and SCSS variables (`colors.module.scss`).

## Database Schema (Key Models)

*   **User:** Handles Customers, Staff, and Admins via `roleId`. Links to `Neighborhood`.
*   **Product:** Inventory items with `stock`, `price`, `category`, and `disabled` status.
*   **Order:** Linked to `User` (Client) and optionally `Staff`. Has `OrderState` and `DeliveryMethod`.
*   **ItemProduct:** Many-to-many relationship between `Order` and `Product` (Order items).
*   **Neighborhood/DeliveryMethod:** For logistics and delivery logic.

## Business Rules (from `requirements.md`)

*   **Store Hours:** 07:50 - 18:00.
*   **Delivery:** Max radius of 3km.
*   **Payment:** Pix, Credit/Debit Card, Cash. No "credit/fiado" allowed.
*   **Refunds:** Must be done physically in-store.

## Building and Running

### Prerequisites
*   Node.js (v18+ recommended)
*   PostgreSQL Database

### Key Commands

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production (includes Prisma generation)
npm run build

# Start production server
npm run start

# Linting
npm run lint

# Database Operations
npm run prisma:generate  # Generate Prisma Client
npm run prisma:seed      # Seed the database
```

## Development Conventions

*   **Strict Typing:** Ensure all new code is strictly typed with TypeScript.
*   **Styling:** Use SCSS Modules for component-level styling. Avoid global CSS where possible.
*   **API Interactions:** Use the service layer (`src/services`) or direct API calls in `src/app/api`.
*   **Authentication:** Protected routes should verify the user's session and role (Admin, Staff, etc.).

---
> Source: [RotomDaPesteBR/el-shadai](https://github.com/RotomDaPesteBR/el-shadai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
