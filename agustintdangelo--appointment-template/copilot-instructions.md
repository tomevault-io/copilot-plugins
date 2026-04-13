## appointment-template

> This repository contains a reusable appointment-booking app template for service businesses.

# AGENTS.md

This repository contains a reusable appointment-booking app template for service businesses.

The first demo vertical is a nail salon, but the app must remain generic enough to adapt to:
- beauty centers
- barbershops
- spas / massage
- tattoo studios
- consultants / session-based services
- any business that sells time slots

## Core product intent
Build a simple, practical booking app with:
- a public booking flow for customers
- an admin area for managing services, staff, schedules, and appointments
- a codebase that is easy to rebrand and extend

## Stack defaults
Prefer these unless there is a strong reason not to:
- Next.js
- TypeScript
- Tailwind
- Prisma
- SQLite for local development
- Zod
- date-fns

Keep the stack simple.
Do not add unnecessary infrastructure.

## Architecture principles
- Keep the domain generic.
- Keep demo content separate from core logic.
- Prefer explicit, maintainable code.
- Avoid overengineering.
- Do not hardcode "nails" into core entities or business logic.
- Favor configuration and seed data over special-case logic.
- Keep business logic outside presentational UI components.

## Domain model guidance
The app will typically revolve around entities like:
- Business
- Service
- StaffMember
- BusinessHours
- StaffAvailability
- BlackoutDate
- Appointment
- AdminUser
- Settings / Branding

These entities should remain reusable across appointment-based verticals.

## Public/customer MVP expectations
The public side should support:
- landing page
- services list
- booking flow
- availability selection
- booking confirmation

## Admin MVP expectations
The admin side should support:
- simple admin auth
- dashboard
- service CRUD
- staff CRUD
- business hours management
- staff schedule management
- blackout dates
- appointment list
- appointment status updates

## Booking logic rules
Availability must be derived from:
- business working hours
- staff working hours
- service duration
- service buffer time if present
- blackout dates
- existing appointments

Avoid fake availability.

## Non-goals for initial versions
Do not add these unless explicitly requested:
- payments
- SMS / WhatsApp integrations
- multi-location support
- SaaS billing
- advanced RBAC
- Google/Outlook sync
- complex analytics
- microservices

## Documentation files
These files are mandatory and must stay updated:
- docs/PRODUCT.md
- docs/ARCHITECTURE.md
- docs/ITERATION_LOG.md
- docs/ADAPTATION_GUIDE.md

## Documentation rule
After every meaningful change, update docs.
At minimum, every meaningful iteration must update:
- docs/ITERATION_LOG.md

## Working style
When implementing features:
1. Read AGENTS.md and docs first
2. Inspect current code before changing architecture
3. Make a short plan before large edits
4. Work in small vertical slices
5. Keep changes cohesive
6. Update docs after the slice is complete
7. Verify before declaring done

## Definition of done
A task is only done when:
- the feature works end-to-end
- code is readable and consistent
- lint passes
- typecheck passes
- tests pass if tests exist
- docs are updated
- no obvious dead code remains

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agustintdangelo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/agustintdangelo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
