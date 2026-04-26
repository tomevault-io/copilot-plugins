## dsm

> A full-stack driver deployment scheduling platform. Replaces manual planning with automated rules for staffing, scoring, billing, and multi-city control.

# FleetFlow DSM — Driver Scheduling & Management System

## Project Overview
A full-stack driver deployment scheduling platform. Replaces manual planning with automated rules for staffing, scoring, billing, and multi-city control.

## Monorepo Structure
```
DSM-main/
├── backend/              ← Node.js + Express + MongoDB + JWT (EMPTY - needs to be built)
├── driver-mobile-app/    ← React Native + Expo (screens scaffolded, logic missing)
├── admin-web-dashboard/  ← React + Vite (3 pages done: Login, DriverApproval, DriverList, Deployments)
```

## Tech Stack
- Backend: Node.js, Express.js, MongoDB (Mongoose), JWT auth
- Mobile: React Native, Expo
- Admin Web: React, Vite, TypeScript

## Current State (what exists)
### Admin Dashboard (admin-web-dashboard/)
- Login page with JWT auth context
- Driver approval page (list pending → approve/reject)
- Driver list page
- Deployment management page
- SidebarLayout component
- API service layer (api.ts, auth.api.ts, drivers.api.ts)

### Driver Mobile App (driver-mobile-app/)
- Auth flow: Welcome, Register, Login screens
- Onboarding: ProfileSetup, DocumentUpload, PendingApproval
- Main tabs: Dashboard, AvailableDeployments (placeholder), Score, Earnings, ProfileSettings
- Navigation structure with AuthStack, OnboardingStack, MainTabs
- API service layer (api.ts, auth.api.ts, driver.api.ts)

### Backend (backend/)
- EMPTY — needs to be built from scratch following the architecture in README

## Backend Architecture to Follow
backend/src/
├── config/         ← env, MongoDB connection, logger
├── models/         ← Mongoose models
├── controllers/    ← HTTP request handlers
├── routes/         ← URL → controller mapping
├── services/       ← Business logic
├── middleware/      ← auth, roles, validation, error handling
└── utils/          ← JWT, geo, time, constants, response helpers

## Core Business Rules (MUST follow exactly)

### Time Slots
Each day has 3 slots: Morning | Midday | Evening
Planning is always per: Location + Date + TimeSlot

### Driver Statuses
under_review → active | restricted | blocked

### Scoring System
- Start: 100 points
- No-show: major deduction (-20)
- Late cancellation (<24h): deduction (-10)
- Completed mission: small bonus (+2)
- 90–100 → priority slot visibility
- 70–89 → normal access
- <70 → only remaining slots
- <60 → restricted status

### Booking Rules
- Max 2 missions per driver per day
- Slot is binding once confirmed
- Driver can cancel: on time (>24h), late (<24h), or no-show

### Confirmation System (automated reminders)
- T-24h, T-12h, T-6h reminders sent to driver
- If not confirmed by T-6h → slot auto-withdrawn, goes back to pool

### Check-in System
- Only valid if: driver within location GPS radius AND enters correct code
- No check-in = no-show

### Overbooking
- If 120 slots needed → release 126–130
- Drivers with lower scores become substitutes when too many show up
- Each city has its own overbooking settings

### Billing Flow
1. System generates mission summary per driver at month end
2. Driver uploads invoice
3. Admin verifies and approves/rejects
4. Payment released after approval

## API Endpoints (build these)
### Auth
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout

### Driver (self)
GET    /api/drivers/me
PUT    /api/drivers/me/profile
POST   /api/drivers/me/documents
GET    /api/drivers/me/documents
POST   /api/drivers/me/submit-application
GET    /api/drivers/me/application-status
GET    /api/drivers/me/score

### Admin → Drivers
GET    /api/admin/drivers/pending
GET    /api/admin/drivers/:id
POST   /api/admin/drivers/:id/approve
POST   /api/admin/drivers/:id/reject
POST   /api/admin/drivers/:id/request-more-documents

### Assignments
GET    /api/assignments/available
POST   /api/assignments/:id/reserve
GET    /api/assignments/my
POST   /api/assignments/:id/confirm
POST   /api/assignments/:id/check-in
POST   /api/assignments/:id/cancel

### Billing
GET    /api/billing/periods
GET    /api/billing/periods/:id
POST   /api/billing/periods/:id/invoice
GET    /api/admin/invoices
POST   /api/admin/invoices/:id/approve
POST   /api/admin/invoices/:id/reject

### Analytics (Admin)
GET    /api/admin/analytics/overview
GET    /api/admin/analytics/no-shows
GET    /api/admin/analytics/cancellations
GET    /api/admin/analytics/locations
GET    /api/admin/analytics/billing

## Coding Rules
- Always use TypeScript
- Use async/await, never callbacks
- All API responses: { success: boolean, data?: any, message?: string }
- Validate inputs with express-validator or zod
- Use HTTP status codes correctly
- Keep existing file structure, don't move or rename existing files
- Add comments for complex business logic
- Backend .env variables: PORT, MONGO_URI, JWT_SECRET, JWT_EXPIRES_IN
```

---

## 🚀 Step 3 — First Prompts to Give Claude Code

Start Claude Code in the project root and give it these prompts **one at a time**:

**Prompt 1 — Build the backend foundation:**
```
Read CLAUDE.md fully. Then build the complete backend in backend/. 
Start with: package.json, src/config/ (db.ts, env.ts, logger.ts), 
src/utils/ (jwt.ts, response.ts, constants.ts, geo.ts), 
and src/middleware/ (auth.middleware.ts, error.middleware.ts, roles.middleware.ts).
Use TypeScript throughout. Follow the architecture in CLAUDE.md exactly.
```

**Prompt 2 — Build the Mongoose models:**
```
Following CLAUDE.md business rules, create all Mongoose models in backend/src/models/:
- User.model.ts (drivers and admins, with status enum)
- Document.model.ts (uploaded driver documents)
- Assignment.model.ts (slots with location, date, timeSlot, overbooking fields)
- Booking.model.ts (driver ↔ assignment, confirmation timestamps, check-in)
- Score.model.ts (driver score history/log)
- BillingPeriod.model.ts and Invoice.model.ts
Include all fields needed to support every business rule in CLAUDE.md.
```

**Prompt 3 — Auth + Driver routes:**
```
Build backend/src/routes/, controllers/, and services/ for:
1. Auth (register, login, logout) with JWT
2. Driver self-service routes (/api/drivers/me/*)
3. Admin driver management (/api/admin/drivers/*)
Follow the API endpoints listed in CLAUDE.md. Connect everything in src/app.ts and src/server.ts.
```

**Prompt 4 — Assignments + Scoring:**
```
Build the assignment and scoring system following CLAUDE.md rules:
- Assignment routes (available, reserve, confirm, check-in, cancel)
- Auto-score changes on: no-show (-20), late cancel (-10), completed (+2)
- Slot visibility filtering based on driver score tier
- T-6h auto-withdrawal logic (can use a cron job or a check on request)
- Overbooking logic per location/timeslot
```

**Prompt 5 — Connect admin dashboard to real API:**
```
The admin-web-dashboard already has pages for driver approval, driver list, and deployment management.
Update admin-web-dashboard/src/services/ to connect to the real backend API at http://localhost:4000.
Then implement the missing admin pages:
- BillingPage (view/approve invoices)
- AnalyticsPage (overview stats, no-show rate, cancellation rate)
Add these routes to App.tsx and add links in SidebarLayout.
```

**Prompt 6 — Flesh out driver mobile app:**
```
The driver mobile app screens are mostly placeholder text.
Implement the real logic for:
- AvailableDeploymentsScreen: fetch /api/assignments/available, show cards with location/time/pay, booking button
- DriverDashboardScreen: show upcoming assignments, confirmation buttons (T-24/T-12/T-6), check-in button
- DriverScoreScreen: show score, tier badge, history
- EarningsScreen: show billing periods, upload invoice
Connect all screens to driver.api.ts service layer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/idrees-raza-mi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
