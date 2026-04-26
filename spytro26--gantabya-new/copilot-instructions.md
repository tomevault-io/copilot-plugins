## gantabya-new

> GoGantabya is a full-stack bus booking platform (Nepal/India) with three portals:

# Gantabya - Bus Booking Platform

## Project Overview
GoGantabya is a full-stack bus booking platform (Nepal/India) with three portals:
- **User portal** — search buses, book tickets, pay via Razorpay (INR) or eSewa (NPR)
- **Admin portal (Plus)** — bus operators manage buses, routes, trips, bookings, offline booking (COD)
- **Super Admin portal** — verify admins, manage platform-wide offers

## Tech Stack
- **Frontend:** React + TypeScript + Vite + Tailwind CSS (in `gantabya_front/`)
- **Backend:** Express + TypeScript + Prisma ORM + PostgreSQL (in `backend_gantabya/`)
- **Payments:** Razorpay (INR), eSewa (NPR), COD (offline admin bookings)

## Project Structure
```
gantabya_front/src/
  App.tsx           — All routes defined here
  config.ts         — API endpoint constants (backend URLs, NOT frontend routes)
  lib/api.ts        — Axios instance with auth interceptor
  pages/            — All page components
  components/       — Shared components (AdminLayout, UserNavbar, etc.)
  utils/currency.ts — Currency formatting (NPR base, INR conversion)

backend_gantabya/src/
  index.ts              — Express app entry
  admin/adminRouter.ts  — All admin API routes
  user/userRouter.ts    — All user API routes
  superadmin/           — Super admin routes
  services/             — Business logic (payments, offers, notifications)
  prisma/schema.prisma  — Database schema
```

## Important Conventions

### Routing
- **Frontend admin routes** use `/plus/*` prefix (e.g., `/plus/dashboard`, `/plus/signin`)
- **Backend admin API routes** still use `/admin/*` prefix (e.g., `/admin/dashboard`, `/admin/signup`)
- **Do NOT confuse frontend routes with API endpoints** — config.ts has API endpoints, App.tsx has frontend routes
- User routes: `/signin`, `/signup`, `/home`, `/book/:tripId`, etc.
- Super admin routes: `/superadmin/*`

### Currency
- Base currency is NPR (Nepali Rupee)
- INR conversion rate: 1 NPR = 0.625 INR (defined in `utils/currency.ts`)
- Offline (COD) bookings can be in either INR or NPR
- Online payments: Razorpay charges INR, eSewa charges NPR

### Authentication
- Admin tokens stored in `localStorage` as `adminToken`
- User tokens stored as `authToken`
- Backend checks cookies first, then `Authorization: Bearer` header
- Admin accounts require email verification + super admin approval

### Database
- Prisma ORM with PostgreSQL
- Key models: User, Bus, Stop, StopPoint, Trip, BookingGroup, Booking, Passenger, Payment, Offer
- Payments have `method` (RAZORPAY/ESEWA/COD), `chargedCurrency` (INR/NPR)

## Build & Run
```bash
# Frontend
cd gantabya_front && npm install && npm run dev

# Backend
cd backend_gantabya && npm install && npm run dev

# Database
cd backend_gantabya && npx prisma migrate dev
```

## Key Notes
- Offline bookings store `codAmount` and `codCurrency` in payment metadata
- Dashboard revenue is split: COD (NPR), COD (INR), Online/Platform revenue
- Admin signup is linked from user signin/signup pages ("Join as Admin")

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spytro26) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
