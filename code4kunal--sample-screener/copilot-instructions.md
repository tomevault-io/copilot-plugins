## sample-screener

> │   ├── auth         # Login, signup, JWT utils

## Backend Structure (.mdc)

### Project Folder Structure:
```
/backend
├── src
│   ├── auth         # Login, signup, JWT utils
│   ├── users        # User profile CRUD, subscription
│   ├── screeners    # Screener logic, execution, backtesting
│   ├── analytics    # Screener performance endpoints
│   ├── cases        # Case studies CRUD
│   ├── payments     # Stripe/Razorpay integrations
│   ├── admin        # Admin tools (invites, dashboard)
│   ├── middlewares  # Auth guards, error handling
│   ├── utils        # Helper functions, API wrappers
│   └── main.ts      # Entry point
├── prisma           # Schema and migrations
├── .env             # Environment variables
└── package.json     # Dependencies
```

### API Structure (REST or GraphQL-based)

- `POST /auth/signup`
- `POST /auth/login`
- `GET /user/profile`
- `PATCH /user/profile`
- `GET /screeners`
- `GET /screeners/:id`
- `POST /screeners/:id/run`
- `POST /screeners/:id/backtest`
- `GET /analytics/screener/:id`
- `GET /cases`
- `GET /cases/:id`
- `POST /payments/checkout`
- `POST /payments/webhook`
- `GET /admin/users`
- `POST /admin/invite`

### ORM & DB Design (PostgreSQL + Prisma)

Entities:
- User
- Screener
- ScreenerRun
- BacktestResult
- CaseStudy
- Subscription
- PaymentTransaction
- AdminInvite

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/code4kunal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
