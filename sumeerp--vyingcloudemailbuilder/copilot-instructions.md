## vyingcloudemailbuilder

> Production email builder SaaS (comparable to Stensul/Dispatch).

# VyingCloud Email Builder

Production email builder SaaS (comparable to Stensul/Dispatch).

## Stack

- **Framework**: Next.js 16 (App Router, Turbopack, TypeScript)
- **Database**: Supabase (Postgres + Auth + RLS)
- **Auth**: Email/password now, OAuth/SSO later
- **AEM**: Adobe Experience Manager OpenAPI Content Fragment Management API
- **Deployment**: Vercel + Cloudflare domain (not yet configured)
- **Repo**: https://github.com/SumeerP/VyingCloudEmailBuilder

## Architecture

### Multi-tenant

Every table is scoped to an `org_id`. RLS policies enforce isolation.
On signup, a trigger auto-creates: org -> user_profile (role=owner) -> default brand_kit.

### AEM Integration

Uses the **AEM OpenAPI Content Fragment Management API** (NOT the deprecated Assets HTTP API).

- Base path: `{AEM_AUTHOR_URL}/adobe/sites/cf/`
- Auth: Adobe IMS OAuth Server-to-Server (`client_credentials` grant)
- Token endpoint: `https://ims-na1.adobelogin.com/ims/token/v3`
- Headers: `Authorization: Bearer {token}`, `X-Api-Key: {clientId}`, `X-Adobe-Accept-Experimental: 1`
- Token cached in-memory with 5-min expiry buffer
- Hibernation detection: HTML responses from AEM are caught and returned as 503 with `AEM_UNAVAILABLE` code
- **PREREQUISITE**: Client ID must be registered via `api.yaml` deployed through Cloud Manager Config Pipeline:
  ```yaml
  kind: "API"
  version: "1"
  data:
    allowedClientIDs:
      author:
        - "<client_id>"
  ```

### Storage Adapter Pattern

The original email builder (~1200 line JSX) used `window.storage.get()` / `window.storage.set()`.
Instead of refactoring the component, we inject a Supabase-backed adapter onto `window.storage`
via `builder-client.tsx`. The adapter routes keys to the correct Supabase tables:
- `"brand_kit"` -> `brand_kits` table
- `"content_blocks"` -> `content_blocks` table
- `"emails:*"` -> `emails` table

## Key Files

```
src/
  components/email-builder.jsx    # Main email builder (modified from original TSX, ~1200 lines)
  app/
    page.tsx                      # Redirects to /builder
    layout.tsx                    # Root layout
    globals.css                   # Minimal reset
    builder/
      page.tsx                    # Server component: fetches user data from Supabase
      builder-client.tsx          # Client wrapper: injects storage adapter, renders EmailBuilder
    auth/
      login/page.tsx              # Email/password login
      signup/page.tsx             # Signup with org name
      callback/route.ts           # OAuth callback handler
    api/aem/
      fragments/route.ts          # CRUD proxy for CF fragments (GET/POST/PATCH/DELETE)
      models/route.ts             # List/get CF models (GET)
      publish/route.ts            # Create or update fragments (POST)
  lib/
    aem/
      auth.ts                     # IMS OAuth token exchange + caching
      client.ts                   # AEM OpenAPI client (models, fragments, variations)
    supabase/
      client.ts                   # Browser Supabase client
      server.ts                   # Server Supabase client
      middleware.ts               # Session refresh middleware
    storage.ts                    # Supabase storage adapter (replaces window.storage)
  middleware.ts                   # Route protection (unauthenticated -> /auth/login)
  types/database.ts               # TypeScript interfaces for all tables
supabase/
  migrations/001_initial_schema.sql  # Full schema with RLS + triggers
```

## Database Tables

- `organizations` - tenants
- `user_profiles` - linked to auth.users, has org_id + role (owner/admin/editor/viewer)
- `emails` - email designs with components JSONB, global_styles, A/B config
- `email_versions` - snapshot history
- `content_blocks` - reusable block library
- `brand_kits` - brand configuration (colors, fonts, spacing, button styles)

## Environment Variables

```
NEXT_PUBLIC_SUPABASE_URL      # Supabase project URL
NEXT_PUBLIC_SUPABASE_ANON_KEY # Supabase anon key
AEM_AUTHOR_URL                # AEM Author instance (https://author-p{id}-e{id}.adobeaemcloud.com)
AEM_IMS_CLIENT_ID             # Adobe Developer Console OAuth S2S client ID
AEM_IMS_CLIENT_SECRET         # Adobe Developer Console OAuth S2S client secret
AEM_IMS_SCOPES                # IMS scopes (AdobeID,openid,aem.folders,aem.fragments.management)
```

## Current Status

- [x] Next.js project setup with TypeScript + Tailwind
- [x] Email builder component integrated (JSX, not TSX)
- [x] Supabase schema with multi-tenant RLS
- [x] Auth pages (login/signup) with auto-provisioning
- [x] AEM OpenAPI proxy routes with IMS OAuth S2S
- [x] AEM hibernation detection
- [x] AEM OpenAPI verified end-to-end (6 models, 48 fragments)
- [x] Pushed to GitHub
- [ ] Supabase project creation (still using placeholder URL)
- [ ] Run migration SQL in Supabase
- [ ] Vercel deployment
- [ ] Cloudflare domain setup
- [ ] Wire email builder CRUD to Supabase (storage adapter exists but needs testing with real Supabase)
- [ ] Test AEM integration end-to-end (env was hibernated last check)

## Build & Dev

```bash
npm run dev          # Start dev server (Turbopack)
npm run build        # Production build
npm run lint         # ESLint
```

## Important Notes

- `email-builder.jsx` is intentionally `.jsx` (not `.tsx`) to avoid TypeScript strict errors on the untyped component
- Builder page uses `export const dynamic = "force-dynamic"` to prevent static prerendering
- The `handle_new_user()` trigger uses `security definer` to bypass RLS during provisioning
- AEM Cloud Service environments hibernate after inactivity; the client detects this and returns 503

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SumeerP) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
