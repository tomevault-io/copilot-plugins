## pocket-watcher

> 🧾 Product Requirements Document (PRD)

🧾 Product Requirements Document (PRD)
Project: Pocket Watcher

Version: 1.0
Date: October 2025
Owner: Arwin Kumar

1. Overview

Purpose:
Pocket Watcher is a modern, lightweight expense tracker designed to help users take control of their personal finances. It allows users to easily log daily expenses, categorize them, and visualize spending trends—all in one clean dashboard.

Goal:
Build a full-stack Next.js web app (App Router) with a Prisma + SQLite backend and NextAuth.js authentication, deployed seamlessly on Vercel.

2. Objectives

Simple and fast expense logging experience.

Category-based analytics with monthly summaries.

Secure user authentication.

Responsive and elegant UI with modern charts.

Lightweight, offline-friendly architecture.

3. Target Users

Individuals managing personal budgets.

Students or freelancers tracking spending.

Users who prefer a private, non-banking-linked tracker.

4. Success Metrics
Metric	Target
Expense entry time	< 10 seconds
Monthly active users	70%
Page load speed	< 2 seconds
Error rate	< 1%
5. Core Features
🧍 Authentication

Email/password & Google sign-in via NextAuth.js

Sessions with secure cookies

💰 Expense Management

Add, edit, delete, and view expenses

Fields: description, category, amount, date, notes

Category filter & search

📊 Dashboard & Analytics

Monthly total + category breakdown (Pie Chart)

Trend chart for weekly/monthly spend

Quick overview cards for totals

🪙 Budget Overview (Phase 2)

User sets monthly spending goal

Progress bar visualization

📤 Export (Phase 3)

Export expenses as CSV or XLSX

6. Tech Stack
Component	Technology
Framework	Next.js 14+ (App Router, TypeScript)
Database	SQLite (via Prisma ORM)
Auth	NextAuth.js
UI	TailwindCSS + shadcn/ui
Charts	Recharts
Deployment	Vercel
7. API Endpoints
Endpoint	Method	Description
/api/auth/*	POST/GET	Auth endpoints via NextAuth
/api/expenses	GET	Fetch user expenses
/api/expenses	POST	Add expense
/api/expenses/:id	PUT	Update expense
/api/expenses/:id	DELETE	Delete expense
8. Prisma Schema
model User {
  id        String   @id @default(cuid())
  name      String?
  email     String   @unique
  password  String?
  expenses  Expense[]
  createdAt DateTime @default(now())
}

model Expense {
  id          String   @id @default(cuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  description String
  category    String
  amount      Float
  date        DateTime @default(now())
  notes       String?
  createdAt   DateTime @default(now())
}

9. UI/UX Structure
Page	Description
/	Landing page with hero, tagline, CTA to sign in
/login & /register	Auth pages
/dashboard	List of expenses + summary charts
/add	Expense form (modal or page)
/settings	Profile and sign-out

Visual Style:

Minimal, modern (inspired by financial dashboards)

Light/Dark mode toggle

Use rounded cards, smooth transitions, icons

10. Roadmap
Phase	Deliverable	Duration
Phase 1	Auth + CRUD + UI	1 week
Phase 2	Charts + Filters + Analytics	1 week
Phase 3	Export + Budget goal	1 week
Phase 4	Polish + Deploy to Vercel	2–3 days

---
> Source: [Arwindpianist/pocket-watcher](https://github.com/Arwindpianist/pocket-watcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
