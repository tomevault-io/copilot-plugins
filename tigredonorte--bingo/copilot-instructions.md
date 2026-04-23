## bingo

> Turborepo monorepo for the Bingo application with card scanning (OCR) capability.

# Bingo Project

Turborepo monorepo for the Bingo application with card scanning (OCR) capability.

## Project Structure

```
apps/
  web/          # Next.js main app (port 3000)
  docs/         # Next.js documentation (port 3001)
packages/
  ui/           # React component library with Storybook
  auth/         # NextAuth v5 authentication
  shared-helpers/  # Prisma + shared utilities
  api-bingo-scanner/ # OCR (Tesseract.js + Sharp)
```

## Development Commands

```bash
pnpm dev           # Run all dev servers
pnpm build         # Build all packages
pnpm lint          # Lint (zero warnings enforced)
pnpm check-types   # TypeScript type checking
pnpm test          # Run tests
```

**Package-specific:**
```bash
pnpm --filter @repo/ui dev        # Storybook only
pnpm --filter @repo/web dev       # Web app only
```

## Database (Prisma)

- Schema: `packages/shared-helpers/prisma/schema.prisma`
- Generate: `pnpm --filter @repo/shared-helpers prisma:generate`
- Migrate: `pnpm --filter @repo/shared-helpers prisma:migrate`
- Studio: `pnpm --filter @repo/shared-helpers prisma:studio`

## Local Development

```bash
docker compose up   # Start PostgreSQL + Redis
```

**Environment:**
```env
DATABASE_URL=postgresql://bingo:bingo_secret@localhost:5432/bingo
AUTH_SECRET=dev-secret
AUTH_URL=http://localhost:3000
SKIP_ENV_VALIDATION=1
```

## Code Quality

- **No `any` types** - use explicit interfaces
- **Zero ESLint warnings** - strict enforcement
- **Strict TypeScript** - enabled in all packages

## Documentation

### UI Development
- **Component guidelines:** `packages/ui/components-guidelines.md`
- **Component catalog:** `packages/ui/components-catalog.md`
- **Per-component docs:** `packages/ui/src/components/<category>/<Component>/<Component>.md`
- **Storybook:** `pnpm --filter @repo/ui dev` (run with `--host 0.0.0.0` for external access)

### Deployment
- **Digital Ocean:** See [DEPLOYMENT.md](./DEPLOYMENT.md)
- **Cloudflare CD:** See `docs/SETUP_CD_INSTRUCTIONS.md`

### General
- **Development guide:** `docs/README.md`

## Deployment

**When to use deployment scripts:**
- When user asks to "run a server", "deploy", or "test on a real server"
- When Claude needs a complete running server (e.g., E2E testing, API testing)
- Use `./scripts/ephemeral/deploy-cloud-init.sh` to create a new droplet

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Runtime | Node.js 18+ |
| Package Manager | pnpm 9.0.0 |
| Frontend | Next.js 16, React 19 |
| Auth | NextAuth 5 |
| UI | MUI + Emotion |
| Database | PostgreSQL + Prisma |
| Cache | Redis |
| Testing | Vitest |

## Environment Credentials - MANDATORY CHECK

**NEVER claim you don't have access to GitHub, Digital Ocean, or Google credentials.**
You have them in environment variables. Before ANY access claim, run:
```bash
env | grep -iE "github|token|digital|ocean|google|api_key"
```

Credentials available:
- `GITHUB_TOKEN` / `GH_PERSONAL_ACCESS_TOKEN` - GitHub API access
- `DO_API_TOKEN` - Digital Ocean API access
- Other service tokens as configured

**For private repo access, use token in URL:**
```bash
git clone https://x-access-token:${GITHUB_TOKEN}@github.com/owner/repo.git
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tigredonorte) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
