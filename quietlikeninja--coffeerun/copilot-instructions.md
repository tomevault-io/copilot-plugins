## coffeerun

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CoffeeRun is a multi‑team coffee ordering web application.
Users belong to one or more teams, create and share daily coffee orders, manage colleagues (including visitors), and view team‑scoped order history and stats.

Deployment: self‑hosted backend via Docker/Dockge; frontend on Vercel.
The application fully supports multi‑team operation. All legacy single‑team assumptions have been removed.

## System Invariants (Do Not Break)
These rules define the core architecture. Any change that violates them is incorrect.
 - **TeamMembership is the sole source of truth for roles**
   - Users do **not** have global roles
   - Roles are scoped per team via `TeamMembership`
 - **All domain data is team‑scoped**
   - Every resource query must include `team_id`
   - Cross‑team access is never allowed
 - **Orders are immutable snapshots**
   - Order items store denormalized drink details at creation time
   - Historical orders must never change when menus or preferences change
 - **Soft deletes are used consistently**
   - Teams, colleagues, and menu items use `is_active = false`
   - Hard deletes are not used for domain data
 - **Auth is team‑aware**
   - JWTs identify the user only
   - Team context and role are resolved server‑side via membership

## Architecture Overview
### Frontend
- Auth uses **email magic links**
- Verification sets an **httpOnly JWT cookie**
- App bootstraps auth state via `/auth/me`
- Users can belong to multiple teams and switch active team
- All API requests are team‑scoped

### Backend
- All routers are mounted under `/api/v1`
- Team‑scoped resources are mounted under `/teams/{team_id}/...`
- Auth middleware resolves:
  - current user
  - current team membership
  - required role (owner / manager / member)

## Deployment Notes
- Backend Docker image is built by GitHub Actions and pushed to GHCR
- Container startup runs migrations before starting the app
- Frontend is deployed on Vercel with SPA rewrites
- Auth cookies are `SameSite=None; Secure` for cross‑origin requests

## Non‑Goals
- Do **not** reintroduce global user roles
- Do **not** wire up the `ADMIN_EMAIL` setting — it is unused legacy config from the single‑team era, pending removal
- Do **not** bypass team scoping for convenience
- Do **not** mutate historical orders
- Do **not** add frontend logic that assumes a single team

---
> Source: [quietlikeninja/coffeerun](https://github.com/quietlikeninja/coffeerun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
