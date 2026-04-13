## cpoint

> A full-stack SvelteKit application for managing equipment rentals and point-of-sale operations with real-time inventory tracking, guide management, and shift-based operations.

# Equipment Rental Management System

A full-stack SvelteKit application for managing equipment rentals and point-of-sale operations with real-time inventory tracking, guide management, and shift-based operations.

**Stack**: SvelteKit, TypeScript, PostgreSQL, Drizzle ORM, Material Web 3
**Deployment**: AWS EC2 instance with systemd service, GitHub Actions deploys on push to `rental-system` (blue-green Docker/Nginx infra exists in repo but is not yet active)
**Key Features**: Equipment rentals (tracked + generic items), guide management with cooldown, store sales, shift-based operations, Excel reporting

## Quick Links

For detailed architecture and module breakdown, see [docs/CODEBASE_MAP.md](docs/CODEBASE_MAP.md).

## Core Architecture

- **Frontend**: Svelte 5 components with Material Design 3 UI (tablet-optimized)
- **Backend**: SvelteKit API routes with PostgreSQL + Drizzle ORM
- **Database**: 13 core tables with JSONB hybrid approach for flexibility
- **Deployment**: EC2 systemd service, auto-deployed via GitHub Actions on push to `rental-system`

## Main Pages

- **Rental Manager** (`/`) - Create rentals, track active/previous, close rentals, manage store sales
- **Login** (`/login`) - Operator authentication with 4-digit passcode
- **Admin Dashboard** (`/admin/*`) - Equipment, guides, operators, products, inventory management

## API Endpoints (14 total)

- Equipment management (CRUD)
- Guide management (CRUD + PIN verification)
- Operator management (CRUD)
- Rental products (bundles with equipment + pricing)
- Rentals (create, close with pricing calculation)
- Shifts (start, end, close with Excel export)
- Store products & sales (POS)
- Health check

## Key Business Logic

- **Pricing**: Hourly (minimum 1 hr, round up) or full-day flat rate
- **Inventory**: Mixed tracked items (unique codes) + generic quantities in single rental
- **Guide Cooldown**: Configurable cooldown after rental return, overridable via PIN
- **Shifts**: Operator-based sessions tracking all rentals and sales
- **Reporting**: Excel export with rental and sales data

## Getting Started

1. Check out rental-system branch (default is set to this)
2. Read [docs/CODEBASE_MAP.md](docs/CODEBASE_MAP.md) for architecture overview
3. Review [RENTAL_SYSTEM_PLAN.md](RENTAL_SYSTEM_PLAN.md) for product specification
4. Check [DEPLOYMENT.md](DEPLOYMENT.md) for deployment instructions

## Development

```bash
npm install
npm run dev        # Start dev server
npm run db:push    # Apply schema migrations
npm run db:studio  # Open database GUI
```

## Production (AWS EC2)

The app runs on an **AWS EC2 instance** — not a managed platform. There is no secrets manager or environment variable injection from CI. Deployed via GitHub Actions on push to `rental-system`:
- Single Node.js process managed by systemd (`rental-app` service)
- App runs on port 3000, health check at `/health`
- Deploy workflow: `git pull` → `npm ci` → `npm run build` → `npm run db:push` → `systemctl restart`
- Blue-green Docker/Nginx infrastructure exists in repo for future use but is not yet active

### Environment Variables / Secrets

Environment variables (e.g. `DATABASE_URL`) live in `.env.production` on the EC2 server at `/opt/rental-system/.env.production`. **When a new env variable or secret is added to the codebase, it must be manually exported on the EC2 server** — there is no automated secret syncing. To add a new secret:

1. SSH into the EC2 instance
2. Edit `/opt/rental-system/.env.production` to add the variable
3. Restart the service: `sudo systemctl restart rental-app`

See [DEPLOYMENT.md](DEPLOYMENT.md) for full server setup details.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheOneWhoBurns)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TheOneWhoBurns)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
