## kcjewellers

> - Backend: Node.js (Express), PostgreSQL (pg-promise/sequelize), Socket.io

# KC Jewellers Project Rules

## Tech Stack
- Backend: Node.js (Express), PostgreSQL (pg-promise/sequelize), Socket.io
- Frontend: Next.js 15 (App Router), Tailwind CSS, Shadcn UI, Framer Motion
- Payments: Razorpay (Smart Collect for RTGS, Subscriptions for SIP)

## Development Philosophy
1. **Preserve Logic:** Never delete the core jewelry estimation logic in `server.js` or `calculationService.js`. Refactor it, don't destroy it.
2. **Mobile First:** All UI components must be designed for mobile screens first (touch targets, readable fonts).
3. **Futuristic UI:** Use Dark Mode, deep purples/golds, and glassmorphism.
4. **Safety:** Always create database migrations before changing schema.

## Key Files
- `server.js`: Main backend entry point.
- `/client`: (To be created) The new Next.js frontend.
- `/services`: Business logic (Estimations, SIP, Live Rates).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GauravSolanki123456789) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
