## gestion-formation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

French-language training management application with two surfaces:
1. **Public Landing Page**: Marketing site where visitors discover and register for formations
2. **Admin Dashboard** (`/admin`): Protected dashboard for managing registrations and session capacity

Core business logic: Each formation session has limited seats. When full, visitors can pre-register for the next week's session.

## Tech Stack

- **Framework**: Next.js 14 (App Router)
- **Database**: Supabase (PostgreSQL with real-time subscriptions)
- **Styling**: Tailwind CSS
- **Notifications**: WhatsApp (Twilio), Email (Resend/SendGrid), Google Sheets API

## Commands

```bash
# Development
npm run dev          # Start development server
npm run build        # Production build
npm run start        # Start production server
npm run lint         # Run ESLint

# Database
npx supabase db push           # Push migrations to Supabase
npx supabase db reset          # Reset and re-seed database
npx supabase functions serve   # Run Edge Functions locally
```

## Project Structure

```
/app
  /page.tsx                    # Landing page
  /admin
    /page.tsx                  # Dashboard overview
    /login/page.tsx            # Admin authentication
/components
  /landing                     # Hero, FormationCards, RegistrationModal, Footer
  /admin                       # StatsCards, FormationsTable, RegistrationsPanel
  /ui                          # Shared UI components
/lib
  /supabase.ts                 # Supabase client configuration
  /notifications.ts            # WhatsApp, Email, Google Sheets integrations
/supabase
  /migrations/001_init.sql     # Database schema
  /seed.sql                    # Initial formations data
  /functions                   # Edge Functions (check_and_register)
```

## Database Schema

Two main tables:
- **formations**: id, slug, titre, description, session_date, max_attendees, current_attendees, mode, is_active, week_number
- **registrations**: id, formation_id, nom, prenom, email, telephone, entreprise, mode_choisi, is_preregistration, status

Critical: Registration must atomically check capacity before inserting. Use Supabase RPC function `check_and_register`.

## Design Requirements

- **Theme**: Dark navy base (#090E1A) with teal accent (#00C896)
- **Fonts**: "Bricolage Grotesque" (headlines), "DM Sans" (body) from Google Fonts
- **All UI text must be in French**
- Mobile-responsive landing page; admin can be desktop-first

## Environment Variables

Required in `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
ADMIN_PASSWORD=
ADMIN_EMAIL=
ADMIN_WHATSAPP_NUMBER=
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
RESEND_API_KEY=
GOOGLE_SHEET_ID=
GOOGLE_SERVICE_ACCOUNT_JSON=
```

## Key Implementation Notes

1. **Seat Counting**: Use database transaction or Supabase RPC to prevent race conditions when registering
2. **Pre-registration**: When `current_attendees >= max_attendees`, set `is_preregistration = true` without incrementing count
3. **Notifications**: All three (WhatsApp, Email, Sheets) trigger on registration; failures in one should not block others
4. **Real-time**: Admin dashboard uses Supabase real-time subscriptions for live updates
5. **Formations**: 4 fixed formations - Power BI, Digital Marketing, Automatisation & Fullstack, Ressources Humaines

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahmeddamouad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
