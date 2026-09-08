## cosmic-speed-draw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development (Rails + Vite together)
bin/dev                   # Start Rails + Vite dev server via foreman (loads .env)
bin/rails server          # Start Rails only (requires .env to be loaded separately)

# Database
bin/rails db:migrate
bin/rails db:test:prepare

# Testing (RSpec)
bundle exec rspec                              # All specs
bundle exec rspec spec/models/user_spec.rb     # Single file
bundle exec rspec spec/models/user_spec.rb:12  # Single example by line

# E2E (Playwright)
npm run e2e               # Builds frontend, starts Rails test server, runs Playwright

# Code quality
bin/rubocop            # Lint
bin/rubocop -a         # Auto-fix
bin/brakeman --no-pager  # Security scan
```

## Architecture

**Rails 8.0 API-only** (`config.api_only = true`) with an embedded Vite/React frontend at `app/client/`.

In development, `FallbackController` proxies all non-API routes to the Vite dev server (`VITE_DEV_URL`, default `http://localhost:5173`). In production it serves the pre-built `public/index.html`. All API routes must be under `/api` to avoid conflict with the SPA catch-all.

### Authentication

Devise + devise-jwt with a dual-token system: a short-lived JWT (15 min) and a long-lived refresh token (30 days).

**Token transport** — `JwtCookieMiddleware` (in `app/middleware/`) sits in the Rack stack and:

- On responses: moves the JWT from the `Authorization` header into an HttpOnly `jwt_token` cookie, and the refresh token into an HttpOnly `refresh_token` cookie.
- On requests: reads the `jwt_token` cookie and writes it back into `HTTP_AUTHORIZATION` so Warden's JWT strategy authenticates normally.

Clients never handle tokens directly — the cookies are HttpOnly and managed transparently.

**Endpoints:**

- Sign up: `POST /api/users`
- Sign in: `POST /api/users/sign_in`
- Sign out: `DELETE /api/users/sign_out`
- Refresh: `POST /api/refresh` — rotates the refresh token and issues a new JWT cookie

**JWT revocation** — `JwtBlacklist` (Denylist strategy) blacklists tokens on sign-out. `Warden::JWTAuth::Middleware` handles blacklisting at the Rack layer, so tokens are revoked even if the controller returns early.

**Important Devise quirk** — `Devise::SessionsController#verify_signed_out_user` uses `warden.user()` which only reads the Warden session. Since `store? false` on the JWT strategy means nothing is ever written to the session, `all_signed_out?` always returns true. `Users::SessionsController` skips this before-action and checks `request.authorization` directly instead.

`Api::AuthController` is the base class for authenticated API endpoints — it runs `before_action :authenticate_user!`. Inherit from it for any controller that requires a logged-in user.

### Database

PostgreSQL. All four database connections in production (`primary`, `cache`, `queue`, `cable` from the default Rails 8 multi-db template) have been collapsed to a single `DATABASE_URL` for Heroku. Solid gem tables (SolidCache, SolidQueue, SolidCable) live in the same database as app tables.

### Infrastructure

Rails 8 solid gems for all persistent infrastructure (no Redis):

- **SolidQueue** — background jobs, runs as a separate `worker` dyno on Heroku
- **SolidCache** — Rails cache store
- **SolidCable** — Action Cable backend

### Environment variables

| Variable                | Required   | Notes                                                     |
| ----------------------- | ---------- | --------------------------------------------------------- |
| `DATABASE_URL`          | Production | Set automatically by Heroku Postgres                      |
| `DEVISE_JWT_SECRET_KEY` | All        | 128-char hex secret, different per environment            |
| `CORS_ORIGIN`           | All        | Single origin; defaults to `http://localhost:5173` in dev |
| `RAILS_MASTER_KEY`      | Production | Decrypts `config/credentials.yml.enc`                     |

Local dev: copy values into `.env` (loaded by foreman via `bin/dev`). The `.env` file is gitignored.

### Testing

RSpec with FactoryBot. Tests hit a real PostgreSQL database — no DB mocking. GitHub Actions CI runs `scan_ruby` (brakeman), `lint` (rubocop), and `test` (rspec) in parallel. The `DEVISE_JWT_SECRET_KEY` secret must be set in GitHub repository secrets.

### Frontend code style

- Always use the `classnames` library (`cn(...)`) for constructing class strings in React components. Prefer it over template literals or string concatenation for all conditional and composed class names.

### Deployment

Heroku with Node.js + Ruby buildpacks (in that order). The Node.js buildpack builds the Vite frontend (`npm run build` at root, which installs client deps with `--include=dev` and copies `app/client/dist/` → `public/`). The `release` dyno runs `bin/rails db:prepare`.

---
> Source: [lovecosma/cosmic-speed-draw](https://github.com/lovecosma/cosmic-speed-draw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
