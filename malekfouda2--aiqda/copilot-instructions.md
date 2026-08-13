## aiqda

> Persistent repo context for future agent turns. Read this first before making changes.

# AGENTS.md

## Purpose
Persistent repo context for future agent turns. Read this first before making changes.

## Project
- Name: `Aiqda`
- Type: full-stack MERN app
- Backend: Node.js + Express + Mongoose
- Frontend: React + Vite + Zustand + Tailwind + Framer Motion
- Payments: Tap subscriptions + webhook sync + recurring renewal support
- Video: Vimeo integration

## Repo Layout
```text
/backend
  /src
    /modules
      /auth
      /users
      /subscriptions
      /payments
      /courses
      /lessons
      /quizzes
      /analytics
      /video
      /instructor-applications
      /studio-applications
    /middlewares
    /utils
    app.js
    server.js
  /tests

/frontend
  /src
    /components
    /pages
    /layouts
    /services
    /store
    /styles
    App.jsx
    main.jsx
```

## Domain Terminology
Frontend display terms differ from DB/backend names:
- Chapter = `course`
- Content = `lesson`
- Creator = `instructor`
- Member = `student`
- Achievement = `certificate`

Do not blindly rename backend fields/routes unless task explicitly requires schema/API change.

## Roles
- `student`: member learning flow
- `instructor`: creator flow
- `admin`: full platform access

## Main Features
- JWT auth with role-based access control
- Chapter/content management
- Quiz system
- Progress analytics
- Tap subscription checkout and webhook activation
- Consultation booking flow
- Instructor application flow
- Studio application flow
- Vimeo video assignment and playback tracking

## Local Run
- Backend port: `3001`
- Frontend port: `5000`
- MongoDB: `mongodb://localhost:27017/aiqda`

Root commands:
```bash
npm run install:all
npm run start
npm run build
npm run test:backend
npm run verify
```

Backend commands:
```bash
cd backend
npm test
npm run dev
npm run seed:demo
npm run seed:consultations
```

Frontend commands:
```bash
cd frontend
npm run dev
npm run build
```

## Environment
Backend expects `.env` values like:
- `PORT`
- `MONGODB_URI`
- `JWT_SECRET`
- `APP_URL`
- `BACKEND_PUBLIC_URL`
- `TAP_SECRET_KEY`
- `TAP_PUBLIC_KEY`
- `SUBSCRIPTION_CURRENCY`
- `VIMEO_ACCESS_TOKEN`
- `SMTP_HOST`
- `SMTP_PORT`
- `SMTP_SECURE`
- `SMTP_USER`
- `SMTP_PASS`
- `EMAIL_FROM`
- `STUDIO_APPLICATION_MEETING_URL`

## Known Product Notes
- `server.js` auto-seeds demo data when DB empty.
- Default admin from seed: `admin@aiqda.com` / `admin123`.
- Consultation types seed independently through `seedConsultationsIfEmpty()`.
- Frontend terminology already rebranded; backend routes/DB names remain original.

## Working Rules For Future Edits
- Preserve existing terminology split between UI and backend.
- Prefer small focused changes; avoid broad refactors unless asked.
- Check both backend and frontend impact for product features.
- When changing subscriptions/payments, watch Tap webhook and renewal paths.
- When changing learning flow, verify quizzes, watch progress, and dashboard states still align.
- When changing admin flows, check sidebar/navigation links too.

## Good First Inspection Targets
- Backend entry: `backend/src/server.js`
- Backend app wiring: `backend/src/app.js`
- Frontend entry: `frontend/src/main.jsx`
- Frontend router/app shell: `frontend/src/App.jsx`
- Frontend API layer: `frontend/src/services`

## Source Of This Context
Based on current repo manifests and `replit.md`. Update this file when architecture or product scope changes.

---
> Source: [malekfouda2/Aiqda](https://github.com/malekfouda2/Aiqda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
