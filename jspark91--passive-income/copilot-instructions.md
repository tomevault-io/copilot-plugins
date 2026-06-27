## passive-income

> This repository is a standalone project for ETHNIC HOUSE.

# AGENTS.md - ETHNIC HOUSE

## Project Identity

This repository is a standalone project for ETHNIC HOUSE.

Do not use assumptions, files, architecture, naming conventions, database schema, routes, or business logic from any previous project. This project is separate from all existing engineering, CAD, tunnel, and internal tooling work.

## Product Goal

Build a web MVP for English-speaking foreigners who want booking support for ETHNIC HOUSE tattoo sessions in Seoul.

The MVP must collect booking requests, artist data, flash design data, and acquisition/conversion data. The primary studio brand is ETHNIC HOUSE in Sillim, Seoul.

## Stack

- Next.js App Router
- TypeScript
- Tailwind CSS
- Supabase
- Vercel
- GA4 and Meta Pixel hooks

## Core Pages

Public:

- `/`
- `/artists`
- `/artists/[slug]`
- `/booking`
- `/faq`
- `/aftercare`
- `/privacy`

Admin:

- `/admin`
- `/admin/leads`
- `/admin/leads/[id]`
- `/admin/artists`
- `/admin/designs`

## MVP Priority

1. Landing page
2. Booking request form
3. Admin lead management
4. Artist list and detail pages
5. Flash design listing
6. UTM tracking
7. Privacy, FAQ, and aftercare pages

## Do Not Do In V1

- Do not create a mobile app.
- Do not add payment integration.
- Do not add real-time chat.
- Do not add nationwide tattoo marketplace features.
- Do not add worker self-signup.
- Do not expose Supabase service role keys to client-side code.
- Do not copy code or structure from unrelated repositories.

## Done Means

Before saying a task is complete:

- Run `npm run lint` when Node is available.
- Run `npm run build` when Node is available.
- Confirm booking form data writes to `booking_requests` when Supabase is configured.
- Confirm admin lead pages can read and update lead status.
- Confirm UTM values are captured.
- Confirm no unrelated project references were introduced.

---
> Source: [jspark91/PASSIVE-INCOME](https://github.com/jspark91/PASSIVE-INCOME) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
