## vsbooking

> This project is a barber shop appointment system built with:

# Project Rules

This project is a barber shop appointment system built with:

- Next.js (App Router)
- TypeScript
- TailwindCSS
- Supabase (Auth + Database)
- Admin panel protected with middleware

## Architecture

Public frontend:
- users can see services
- users can book appointments
- users can cancel appointments with time restriction

Admin panel:
- route: /admin
- protected via middleware
- access allowed only if email exists in `admins` table in Supabase

Authentication:
- Google OAuth via Supabase
- unauthorized users must receive 404 when trying to access /auth or /admin

## Security rules

- middleware must protect:
  - /admin
  - /admin/*
  - /auth
- check email against `admins` table
- unauthorized users return 404 page

## Appointments logic

Appointments can be cancelled only if:

current_time < appointment_time - cancelation_limit_hours

The cancelation limit must be configurable from the admin panel.

## UI Guidelines

UI style must be:

- Apple-like minimal
- smooth animations
- glass effect cards
- subtle shadows
- modern spacing

Sidebar must have smooth transitions when switching sections.

Admin actions must show toast notifications:

- admin added successfully
- admin removed successfully
- appointment cancelled
- appointment created

Use modern UI patterns and keep the interface clean and minimal.

Never break existing functionality.

---
> Source: [viaggiostyle-netizen/vsbooking](https://github.com/viaggiostyle-netizen/vsbooking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
