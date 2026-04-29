## california-hair-surgeon

> Project overview, tech stack, folder structure, and key constraints for Surg Assist


# Surg Assist — Project Overview

A lightweight practice management tool for small hair surgery centers. Keep everything simple — this is NOT a complex enterprise app.

## Tech Stack
- **Hosting**: Google Cloud
- **Database**: MongoDB Atlas
- **Backend**: Node.js + Express (`/server/index.js`)
- **Parse Server**: Isolated entirely to `/server/parse.js` — no Parse logic anywhere else
- **Frontend**: React JS (`/client`)
- **UI Framework**: MUI (Material UI)

## Folder Structure
```
/client/src
  /layouts        ← AuthLayout, AccountLayout, AdminLayout, RemoteLayout
  /pages
    /auth         ← Login, Signup, ForgotPassword
    /admin        ← Accounts, GlobalDefaults
    /account      ← Home, Surgeries, SurgeryDetail, Technicians, Settings
    /remote       ← SurgeryPicker, CountingInterface
  /components     ← shared reusable components only
  /api            ← all REST calls in one place
  /hooks
  /theme
    tokens.js     ← raw design tokens (only place hex values live)
    index.js      ← MUI createTheme() adapter

/server
  index.js        ← Express app, routes, middleware
  parse.js        ← ALL Parse Server config/logic isolated here
  /routes
  /middleware
    auth.js
    roles.js
```

## Key Constraints
- Frontend talks to backend via **standard REST API only** — no Parse SDK on the client
- Parse Server is isolated to `parse.js` so it can be removed without touching the rest of the app
- Keep the codebase small and simple — avoid over-engineering
- Before building any new component, search `/components` for something that already exists

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlpineDesign) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
