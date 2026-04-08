## um25

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**ULTIMA MILLA Corporate Website** - Production Astro SSR application with Directus headless CMS backend, serving www.ultimamilla.com.ar and related services.

**Critical**: This is a PRODUCTION system serving 5 live websites with 24/7 uptime requirements. Follow strict server architecture rules documented in `.windsurf/rules/arquitectura-servidor-reglas.md`.

---

## Development Commands

### Essential Commands

```bash
# Development
npm run dev                    # Start dev server (localhost:4321)
npm run build                  # Production build
npm run preview                # Preview production build

# Testing
npm test                       # Run Jest tests
npm run test:watch             # Watch mode
npm run test:coverage          # Coverage report
npm run test:ci                # CI environment tests

# Code Quality
npm run lint                   # ESLint check
npm run lint:fix               # Auto-fix lint issues

# Image Processing
npm run process-images         # Process images for optimization
npm run optimize-images        # Optimize existing images
npm run prebuild               # Runs before build (includes process-images)

# Database & API Validation
npm run validate:database      # Validate database connectivity
npm run validate:images        # Validate image loading
npm run validate:api           # Test API connectivity
npm run test:migration         # Full migration validation suite
```

### Production Deployment

```bash
# NEVER deploy manually to production
# All deployments MUST go through Git Flow → GitHub Actions

# Correct workflow:
git checkout develop
git pull origin develop
git checkout -b feature/your-feature
# ... make changes ...
git commit -m "feat: your feature"
git push origin feature/your-feature
# Create PR to develop, then PR to master for deployment
```

---

## Critical Architecture Rules

### Production Baseline

**Baseline Version**: `v0.0.1-production-baseline` (tag)
- This is the immutable source of truth
- Production server at 23.105.176.45
- NEVER modify production server directly

### Git Flow Workflow (MANDATORY)

```
master (production) ← Protected, requires PR
  └── develop (integration) ← Base for features
      └── feature/* ← Your work here
```

### Prohibited Actions

**NEVER**:
- Push directly to `master`
- Edit files directly on production server
- Run `git pull` manually on server
- Modify `.env` files in production
- Execute `npm install` on server without CI/CD
- Disable monitoring cron jobs
- Delete logs without rotation

### Emergency Protocol

If production site is down:
```bash
ssh ultimamilla
pm2 restart astro-ultimamilla

# If that fails:
cd /root/fumbling-field
git checkout v0.0.1-production-baseline
pm2 restart astro-ultimamilla
```

**Full rules**: See `REGLAS_ARQUITECTURA_SERVIDOR.md` and `.windsurf/rules/arquitectura-servidor-reglas.md`

---

## Architecture Overview

### Tech Stack

```yaml
Frontend:
  Framework: Astro 5.7.4 (SSR mode)
  Styling: Tailwind CSS
  Icons: Lucide, Iconify
  Interactive: Alpine.js
  Process: PM2 (process: astro-ultimamilla)

Backend:
  CMS: Directus 10.8.3
  Database: PostgreSQL 15 (Docker)
  Cache: Redis 7 (Docker)

Web Server:
  Proxy: Nginx (reverse proxy)
  Ports: 80/443 (HTTP/HTTPS)
  SSL: Let's Encrypt

Process Management:
  PM2: astro-ultimamilla (port 4321)
  Node: 18.x
```

### Data Flow

```
Client Request
    ↓
Nginx (:80/:443)
    ↓
PM2: astro-ultimamilla (:4321)
    ↓
Directus API (internal Docker network)
    ↓
PostgreSQL (:5432) + Redis (:6379)
```

### Directus Integration

**Connection Pattern**:
```typescript
// src/lib/directus.ts
import { createDirectus, rest } from '@directus/sdk';

const directus = createDirectus(
  process.env.PUBLIC_DIRECTUS_URL || 'http://localhost:8055'
).with(rest());

// Usage in pages:
import { directus } from '@/lib/directus';
import { readItems } from '@directus/sdk';

const items = await directus.request(
  readItems('collection_name', {
    filter: { status: { _eq: 'published' } },
    fields: ['*', 'relations.*']
  })
);
```

**Key Collections**:
- `antecedentes` - Case studies/portfolio items (469 items)
- `Servicios` - Services offered
- `Imagenes` - Image assets linked to antecedentes
- `sectores` - Industry sectors for filtering

### Image Handling

**Critical**: Images come from Directus and require special handling.

```typescript
// src/utils/directus.js - getImageUrl()
// Priority: Directus URL > UUID fallback > placeholder

// src/utils/imageFixer.js
// Maps 13 known broken images to working URLs
// Used in sector pages (constructoras, bodegas, salud, etc.)
```

**Image Components**:
- `EnhancedImage.astro` - Standard image with fallback
- `LazyImage.astro` - Lazy loading implementation
- `OptimizedImage.astro` - Sharp-optimized images

### Sector Filtering

Sector pages (constructoras, bodegas, salud, aeropuertos, software, gobiernosectorpublico) use **positive filtering**:

```typescript
// Filter by specific keywords in title/description
const keywords = ['keyword1', 'keyword2', 'keyword3'];
const filteredItems = items.filter(item =>
  keywords.some(kw =>
    item.Nombre?.toLowerCase().includes(kw) ||
    item.Descripcion?.toLowerCase().includes(kw)
  )
);
```

---

## Project Structure

```
/
├── src/
│   ├── components/          # Astro components
│   │   ├── antecedentes/   # Case study components
│   │   ├── common/         # Shared components (EnhancedImage, etc.)
│   │   └── SEO/            # SEO components
│   ├── layouts/            # Page layouts
│   │   └── Layout.astro   # Main layout (includes Analytics, SEO)
│   ├── pages/              # Routes (file-based routing)
│   │   ├── index.astro    # Homepage
│   │   ├── servicios/     # Services pages
│   │   ├── antecedentes/  # Case studies
│   │   │   └── [id]/[slug].astro  # Dynamic routes
│   │   └── *.astro        # Sector pages
│   ├── lib/               # Utilities
│   │   └── directus.ts    # Directus client
│   └── utils/             # Helper functions
│       ├── directus.js    # Image URL helpers
│       └── imageFixer.js  # Broken image mappings
├── public/                # Static assets
│   ├── uploads/           # Directus image uploads
│   └── favicon.svg        # Favicon
├── .github/workflows/     # CI/CD pipelines
│   ├── production-deploy.yml  # Auto-deploy on master
│   └── pr-checks.yml      # PR validation
├── scripts/               # Build and deployment scripts
│   ├── health-check.sh    # Production health checks
│   └── server-metrics.sh  # Server monitoring
└── directus-admin/        # Directus Docker setup
```

---

## Key Configuration Files

### Environment Variables

**Development** (`.env.local`):
```bash
PUBLIC_DIRECTUS_URL=http://localhost:8055
NODE_ENV=development
```

**Production** (`.env` - NEVER commit):
```bash
PUBLIC_DIRECTUS_URL=https://admin.ultimamilla.com.ar
DATABASE_URL=postgresql://...
DIRECTUS_KEY=...
DIRECTUS_SECRET=...
```

### Astro Config

`astro.config.mjs`:
- **Output**: `server` (SSR mode)
- **Adapter**: `@astrojs/node` (standalone)
- **Port**: 4321
- **Alias**: `@` → `/src`

### PM2 Config

`ecosystem.config.js`:
```javascript
{
  name: 'astro-ultimamilla',
  script: './dist/server/entry.mjs',
  instances: 1,
  exec_mode: 'fork',
  env: { NODE_ENV: 'production', PORT: 4321 }
}
```

---

## Development Workflow

### 1. Starting Development

```bash
# Clone repo
git clone https://github.com/martinsantos/um25.git
cd fumbling-field

# Install dependencies
npm ci

# Start dev server
npm run dev
# → http://localhost:4321
```

### 2. Working with Directus Locally

Directus runs in Docker (port 8055):
```bash
cd directus-admin
docker-compose up -d

# Access admin panel:
# http://localhost:8055
# Credentials in .env file
```

### 3. Adding New Pages

```typescript
// src/pages/new-page.astro
---
import Layout from '@/layouts/Layout.astro';
import { directus } from '@/lib/directus';
import { readItems } from '@directus/sdk';

// Fetch data from Directus
const data = await directus.request(
  readItems('collection_name')
);
---

<Layout title="Page Title">
  <!-- Your content -->
</Layout>
```

### 4. Image Optimization

```bash
# Before build, process images
npm run process-images

# Build includes prebuild hook
npm run build  # Automatically runs process-images
```

---

## Testing

### Test Structure

```javascript
// __tests__/component.test.js
import { expect, test } from '@jest/globals';

test('component renders correctly', () => {
  // Test implementation
});
```

### Running Tests

```bash
# Single run
npm test

# Watch mode (for TDD)
npm run test:watch

# Coverage report
npm run test:coverage

# CI environment
npm run test:ci
```

---

## Monitoring & Health Checks

### Automated Monitoring

Production server runs cron jobs:
```bash
# Health checks every 5 minutes
*/5 * * * * /root/scripts/health-check.sh

# Server metrics every hour
0 * * * * /root/scripts/server-metrics.sh
```

### Health Check Logs

```bash
# SSH to production
ssh ultimamilla

# View health check logs
tail -f /var/log/health-check.log

# View server metrics
/root/scripts/server-metrics.sh

# PM2 status
pm2 list
pm2 logs astro-ultimamilla
```

### Services Monitored

```
✅ www.ultimamilla.com.ar
✅ sgi.ultimamilla.com.ar
✅ www.umbot.com.ar
✅ viveroloscocos.com.ar
✅ PM2 processes
✅ PostgreSQL Docker container
```

---

## CI/CD Pipeline

### GitHub Actions Workflows

**`production-deploy.yml`** (triggers on push to `master`):
1. Build & test
2. Deploy via rsync to server
3. Restart PM2: `astro-ultimamilla`
4. Health check
5. Notification

**`pr-checks.yml`** (triggers on PRs):
1. Lint validation
2. Build verification
3. Tests
4. Auto-comment with results

### Required GitHub Secrets

```
SSH_PRIVATE_KEY    # For deployment
```

---

## Common Issues & Solutions

### Issue: Images Not Loading

**Solution**: Check Directus URL configuration and image UUID mapping in `imageFixer.js`

```typescript
// src/utils/imageFixer.js
export const imageFixMap = {
  'broken-image-id': 'working-url',
  // Add mappings here
};
```

### Issue: PM2 Process Crashed

```bash
ssh ultimamilla
pm2 restart astro-ultimamilla
pm2 save
```

### Issue: Directus API Not Responding

```bash
# Check Docker containers
docker ps | grep postgres

# Restart if needed
cd /root/fumbling-field/directus-admin
docker-compose restart
```

### Issue: Build Failures

```bash
# Clear cache and rebuild
rm -rf dist/ .astro/
npm ci
npm run build
```

---

## Important Documentation

**Must Read Before Making Changes**:
1. `REGLAS_ARQUITECTURA_SERVIDOR.md` - Complete server rules
2. `.windsurf/rules/arquitectura-servidor-reglas.md` - Quick reference
3. `WORKFLOW_GITFLOW.md` - Git Flow workflow
4. `MONITORING_SETUP.md` - Monitoring configuration
5. `IMPLEMENTATION_COMPLETE.md` - System overview

**Architecture Docs**:
- `ARQUITECTURA_DIRECTUS_BACKEND.md` - Directus integration details
- `BASELINE_PRODUCTION_SYNC_REPORT.md` - Production baseline report

---

## Deployment Checklist

Before merging to master:

- [ ] Code builds successfully (`npm run build`)
- [ ] Tests pass (`npm test`)
- [ ] Lint passes (`npm run lint`)
- [ ] Tested locally with production build (`npm run preview`)
- [ ] PR approved and merged to `develop` first
- [ ] Directus schema changes deployed (if any)
- [ ] Environment variables updated (if needed)
- [ ] Backup exists (automated via CI/CD)

---

## Contact & Support

**Production Server**:
- IP: `23.105.176.45`
- SSH: `ssh ultimamilla` (configured in `~/.ssh/config`)

**Repository**:
- GitHub: https://github.com/martinsantos/um25
- Baseline Tag: `v0.0.1-production-baseline`

**Services**:
- Main: https://www.ultimamilla.com.ar
- Admin: https://admin.ultimamilla.com.ar
- SGI: https://sgi.ultimamilla.com.ar

---

## Final Notes

### Golden Rules

1. **Baseline is Sacred**: `v0.0.1-production-baseline` is immutable
2. **Git Flow is Law**: feature → develop → master (no exceptions)
3. **Production is Read-Only**: Only CI/CD writes to production
4. **Backup Before Change**: No backup = no change
5. **Rollback Over Fix**: In emergencies, rollback first, fix later

### Never Assume

- Always verify changes locally first
- Test with production build before deploying
- Check logs after deployment
- Monitor health checks for 30 minutes post-deploy

**This is a production system with zero-downtime requirements. When in doubt, ask for review before proceeding.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martinsantos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/martinsantos)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
