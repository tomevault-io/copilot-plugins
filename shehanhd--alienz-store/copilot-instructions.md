## alienz-store

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

Monorepo with a FastAPI backend (root) and a React/Vite frontend (`frontend/`).

```
/
├── api/              # FastAPI application package
│   ├── index.py      # App factory, middleware, router registration
│   ├── config.py     # Pydantic settings (reads .env)
│   ├── db.py         # psycopg2 RealDictCursor connection dependency
│   ├── auth.py       # JWT creation/decoding, password hashing
│   ├── dependencies.py  # FastAPI deps: get_current_user, require_admin, require_owner
│   ├── middleware.py  # MaintenanceModeMiddleware
│   └── routers/      # One file per domain (auth, products, images, categories, account, enquiries, etc.)
├── migrations/       # Raw SQL files, run manually with psql
├── tests/            # pytest tests
├── frontend/         # React + Vite + TypeScript
│   └── src/
│       ├── api/      # Axios client + Zod-validated fetch functions per domain
│       ├── components/  # ui/ (Button, Input, Spinner, ImageUploader, etc.) + layout/ + guards/
│       ├── contexts/    # AuthContext (access token state, refresh logic), ToastContext, ConfirmContext
│       ├── hooks/       # useAuth, useDragSort
│       ├── pages/       # admin/, auth/, public/, account/
│       ├── styles/      # global.css, variables.css (CSS custom properties)
│       ├── types/       # index.ts — shared TypeScript interfaces
│       └── utils/       # compressImage.ts
└── vercel.json       # All requests rewrite to /api/index (Vercel serverless)
```

## Commands

### Backend
```bash
# Install dependencies
pip3 install -r requirements.txt

# Run dev server (from repo root)
uvicorn api.index:app --reload

# Run tests
pytest

# Run a single test file
pytest tests/test_auth.py

# Run tests without DB
pytest -m no_db
```

### Frontend
```bash
cd frontend

npm run dev          # Vite dev server (localhost:5173)
npm run build        # tsc + vite build
npm run test         # vitest watch mode
npm run test:run     # vitest single run
npm run test:coverage
```

### Database migrations
Migrations are plain SQL files, run manually in order:
```bash
psql $DATABASE_URL -f migrations/001_initial.sql
psql $DATABASE_URL -f migrations/002_ref_colors_sizes.sql
psql $DATABASE_URL -f migrations/003_ref_colors_hex.sql
psql $DATABASE_URL -f migrations/004_products_is_featured.sql
psql $DATABASE_URL -f migrations/005_products_category_ids.sql
```

### First-time setup (owner account)
```bash
curl -X POST http://localhost:8000/setup \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com","password":"...","first_name":"...","last_name":"..."}'
```

## Architecture

### Auth flow
- Login → backend sets HttpOnly `refresh_token` cookie + returns `access_token` in JSON body
- Frontend stores access token in React state (via `AuthContext`) and an `accessTokenRef` for the Axios interceptor
- Every request sends `Authorization: Bearer <access_token>`
- On 401, the Axios response interceptor calls `/auth/refresh` (sends the cookie automatically due to `withCredentials: true`), gets a new access token, and retries the original request once
- The interceptor skips the retry logic if the failing request IS `/auth/refresh` (prevents infinite loop)
- `secure` on the refresh cookie is `false` in development, `true` in production — controlled by `settings.environment`

### Backend patterns
- All DB access uses `psycopg2` with `RealDictCursor`; the `get_db` dependency yields a connection that auto-commits on success and rolls back on exception
- Every product endpoint that returns data must explicitly convert psycopg2 types to JSON-safe Python primitives (`str()` for UUIDs, `float()` for Decimal price, `.isoformat()` for datetimes). Use `_normalise_product_row()` in `products.py` for list endpoints, and do it manually for detail endpoints.
- Role hierarchy: `client < admin < owner`. Guards: `require_admin` (admin or owner), `require_owner` (owner only)
- Image upload goes to Supabase Storage via `api/storage.py`; images are processed (resize + WebP conversion) via `api/image_processor.py` before upload
- Site-wide settings (max products, image limits, maintenance mode, etc.) live in `site_config` DB table; read at request time via `get_config(key, conn)`
- `account.py` covers: `GET/PUT /account/profile`, `POST /account/change-password`, `GET/PUT /account/address`, `GET /account/enquiries`. All profile/user endpoints must return `is_active` — omitting it causes the frontend Zod `UserSchema` to reject the response.

### Frontend patterns
- All API calls go through `frontend/src/api/client.ts` → `getApiClient()` returns the singleton Axios instance
- Every API module (`products.ts`, `auth.ts`, etc.) parses responses with Zod schemas from `api/schemas/`. Schemas use `.default()` and `.passthrough()` to tolerate partial responses from list endpoints
- CSS is CSS Modules (`.module.css`) per component. Design tokens live in `variables.css` (`--font-heading`, `--font-body`, `--text-primary`, `--gold`, `--border`, `--space-*`, etc.)
- Admin pages are gated by `RequireAdmin` route guard; owner-only routes additionally wrapped in `RequireOwner`
- Drag-and-drop reordering uses `useDragSort` hook (HTML5 DnD, no library); calls a `PATCH .../reorder` endpoint on drop
- Toast notifications via `ToastContext` / `useToast` hook. Confirm dialogs via `ConfirmContext` / `useConfirm` hook. Both are mounted in `App.tsx`.
- Body font is **Nunito** (`--font-body`); heading font is set via `--font-heading`. Loaded via Google Fonts in `index.html`.
- Product images use `border-radius: 12px` on the image wrapper (not the card). `ImageGallery` main image uses `border-radius: 12px`, thumbnails use `border-radius: 8px`.

### Key env vars (`.env` at repo root)
```
DATABASE_URL=
JWT_SECRET=
ENVIRONMENT=development   # set to "production" to enable Secure cookie
FRONTEND_URL=http://localhost:5173
SUPABASE_URL=
SUPABASE_SERVICE_KEY=
SUPABASE_STORAGE_BUCKET=product-images
```

Frontend reads `VITE_API_URL` (defaults to `http://localhost:8000`).

---
> Source: [ShehanHD/alienz-store](https://github.com/ShehanHD/alienz-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
