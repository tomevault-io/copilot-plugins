## tarmac

> Core Tarmac application context (always applied)


# Tarmac Application

Mobile-first Symfony webapp for aviation club management using Tabler.io.

## Tech Stack
- **Backend**: Symfony 7.x (PHP 8.3+)
- **Frontend**: Tabler.io + Stimulus.js
- **Database**: PostgreSQL
- **Translations**: French (FR) - ICU format required

## Key Architecture
- Multi-tenant: subdomain-based clubs (e.g., `demo.tarmac.wip`)
- Role-based access: Member, Manager, Inspector, Admin
- Mobile-first responsive design

## Quick Commands
- Start server: `bin/server`
- Clear cache: `symfony console cache:clear`
- Extract translations: `symfony console translation:extract --force --format=xlf fr`

## Important Notes
- Local dev domain: `tarmac.wip` (requires Symfony proxy)
- All user-facing strings must use translation keys in camelCase
- Mobile-first design: start with 320px, enhance for 768px+

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tilotiti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
