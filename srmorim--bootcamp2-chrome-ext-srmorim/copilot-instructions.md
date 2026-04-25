## bootcamp2-chrome-ext-srmorim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Bootcamp Pomodoro PWA** - Progressive Web App with Node.js backend implementing a Pomodoro timer with session history, statistics, and motivational quotes. Built as part of Bootcamp II final delivery.

**Version:** 2.0.0

**Stack:**
- Frontend: Vite + Vanilla JS + PWA Plugin
- Backend: Node.js + Express
- Testing: Playwright (E2E)
- Containerization: Docker + Docker Compose
- CI/CD: GitHub Actions + GitHub Pages

## Project Type

Progressive Web App (PWA) with RESTful API backend - Monorepo with workspaces

## Development Commands

### Setup

```bash
# Install all dependencies (root + apps)
npm install

# Or install individually
cd apps/web && npm install
cd apps/api && npm install
cd tests/e2e && npm install && npx playwright install chromium
```

### Development

```bash
# Run web dev server (Vite HMR on port 8080)
npm run dev:web

# Run API dev server (Node watch mode on port 3000)
npm run dev:api

# Run both in separate terminals for full development
```

### Build & Preview

```bash
# Build web PWA for production
npm run build:web

# Preview production build
cd apps/web && npm run preview
```

### Testing

```bash
# Run all E2E tests (requires web + API running)
npm run test:e2e

# Run API unit tests (16 tests - sessions + quotes)
npm run test:unit
# Or directly:
cd apps/api && npm test

# Run E2E tests with UI (interactive mode)
cd tests/e2e && npm run test:ui

# Run E2E tests in headed mode (see browser)
cd tests/e2e && npm run test:headed

# View Playwright report
cd tests/e2e && npm run report
```

### Lighthouse CI

```bash
# Run Lighthouse CI locally
npx @lhci/cli@0.12.x autorun --config=.lighthouserc.cjs

# Requires web server running on localhost:8080
```

### Docker

```bash
# Build and start all services
docker-compose up --build

# Or use npm scripts
npm run docker:up

# Stop services
npm run docker:down

# View logs
npm run docker:logs

# Access:
# - Web PWA: http://localhost:8080
# - API: http://localhost:3000
```

## Architecture

### Monorepo Structure

```
apps/
├── web/          # Frontend PWA (Vite)
│   ├── src/
│   │   ├── main.js        # UI initialization & event handlers
│   │   ├── timer.js       # Timer class (localStorage state)
│   │   ├── api-client.js  # API communication layer
│   │   └── styles.css
│   ├── public/icons/
│   ├── vite.config.js     # PWA manifest + service worker config
│   └── Dockerfile
│
├── api/          # Backend API (Express)
│   ├── src/
│   │   ├── index.js              # Express app & routes
│   │   ├── routes/sessions.js    # Sessions CRUD endpoints
│   │   └── services/quotes.js    # Quotable API proxy
│   └── Dockerfile
│
tests/e2e/        # E2E tests (Playwright)
├── pwa.spec.ts   # PWA functionality tests
└── api.spec.ts   # API endpoint tests

docker-compose.yml  # Multi-container orchestration
```

### Communication Flow

```
Browser PWA ←→ API Client ←→ Express API ←→ In-Memory Store
     ↓              ↓              ↓
localStorage   Service Worker  Quotable API
```

### State Management

**Frontend (apps/web/src/timer.js):**
- Timer state persisted to `localStorage` (mode, timeRemaining, isRunning, sessionsCompleted)
- Settings persisted to `localStorage` (durations, sound, notifications)
- `Timer` class manages state + tick interval + callbacks

**Backend (apps/api/src/routes/sessions.js):**
- Sessions stored in-memory array (resets on server restart)
- Simple CRUD operations with validation
- Statistics aggregation on-demand

### PWA Configuration

**Service Worker (apps/web/vite.config.js):**
- Workbox with `autoUpdate` strategy
- Caches: static assets, API responses (NetworkFirst), quotes API
- Offline support with cached fallbacks

**Manifest:**
- `base: '/bootcamp2-chrome-ext-SrMorim/'` for GitHub Pages deployment
- Standalone display mode
- Icons: 192x192 and 512x512 (maskable)

## API Endpoints

**Base URL:** `http://localhost:3000`

- `GET /api/health` - Health check
- `GET /api/quote` - Get random motivational quote (proxied from Quotable)
- `GET /api/sessions` - List all sessions (sorted by completedAt desc)
- `GET /api/sessions/:id` - Get specific session
- `GET /api/sessions/stats` - Get statistics (total, today, thisWeek, byMode)
- `POST /api/sessions` - Create session (body: `{mode, duration, completedAt}`)
- `DELETE /api/sessions/:id` - Delete session
- `DELETE /api/sessions` - Clear all sessions

**Validation:**
- `mode` must be: `focus`, `shortBreak`, or `longBreak`
- `duration` must be positive number
- Returns 400 for validation errors, 404 for not found, 500 for server errors

## Important Files & Patterns

### Frontend Entry Point (apps/web/src/main.js)

Main UI controller that:
- Initializes `Timer` instance
- Handles button clicks (start, pause, reset, skip, mode switching)
- Updates DOM every second (timer display, mode UI, session counter)
- Fetches and displays motivational quotes
- Manages session history view and statistics
- Calls `apiClient` to persist completed sessions

### Timer Logic (apps/web/src/timer.js)

Timer class with:
- State persistence via localStorage
- Tick callbacks (called every second when running)
- Complete callbacks (called when timer reaches 0)
- Auto-mode-switching based on session count
- Sound/notification support

### API Client (apps/web/src/api-client.js)

Abstraction layer for API calls:
- Configurable `API_URL` (defaults to localhost:3000 or window.API_URL)
- Error handling with fallbacks
- Methods: `createSession()`, `getSessions()`, `getStats()`, `getQuote()`

### Docker Configuration

**apps/api/Dockerfile:**
- Base: node:20-alpine
- Runs `npm start` (production mode)
- Exposes port 3000

**apps/web/Dockerfile:**
- Multi-stage: build with node:20-alpine, serve with nginx:alpine
- Copies dist/ to nginx html directory
- Exposes port 80 (mapped to 8080 in docker-compose)

**docker-compose.yml:**
- Two services: `api` and `web`
- Health checks on both services
- `web` depends on `api` health check
- Custom bridge network `pomodoro-network`
- API_URL build arg for web service

## CI/CD Pipeline

**File:** `.github/workflows/ci-pwa.yml`

**Jobs:**
1. **build-test**: Install deps → Run API unit tests → Build web → Start API & web preview → Run E2E tests → Run Lighthouse CI (with enforcement) → Upload artifacts
2. **deploy**: Build web with production API_URL → Upload as Pages artifact → Deploy to GitHub Pages via official Actions (on push to main)
3. **docker-build**: Test Docker builds for both services → Validate docker-compose config

**Artifacts:**
- `playwright-report/` - HTML test report (E2E)
- `test-results/` - JSON results (E2E)
- `lighthouse-report/` - Lighthouse CI results + assertions
- `web-dist/` - Built PWA

**Quality Gates:**
- ✅ API unit tests must pass (16 tests)
- ✅ E2E tests must pass (16 tests)
- ✅ Lighthouse scores ≥ 80 (enforced by .lighthouserc.cjs)

**Environment Variables:**
- `E2E_BASE_URL` - Base URL for E2E tests (default: http://localhost:8080)
- `API_URL` - API base URL for API tests (default: http://localhost:3000)
- `VITE_API_URL` - Build-time API URL for production (GitHub Pages uses Render backend)
- `LHCI_GITHUB_APP_TOKEN` - Optional Lighthouse CI GitHub App token

## Testing Strategy

### Unit Tests (apps/api/src/)

**Sessions Routes (sessions.test.js) - 12 tests:**
- POST /api/sessions: creates valid sessions, rejects invalid data
- GET /api/sessions: returns all sessions sorted correctly
- GET /api/sessions/stats: calculates statistics (total, today, thisWeek, byMode)
- DELETE /api/sessions: clears all sessions
- Validates: mode (focus/shortBreak/longBreak), duration (positive number), required fields

**Quotes Service (quotes.test.js) - 4 tests:**
- getRandomQuote: returns quotes with required fields (content, author, tags)
- Fallback mechanism: always returns valid quote even on API failure
- Integration tests with real Quotable API calls

**Running:** `cd apps/api && npm test` (uses Node.js test runner)

### E2E Tests (tests/e2e/)

**PWA Tests (pwa.spec.ts) - 8 tests:**
- Homepage loads with correct title and header
- Manifest is valid and accessible at `/manifest.webmanifest`
- Service worker registers successfully
- Timer controls work (start, pause, reset, skip)
- Mode switching works (focus, shortBreak, longBreak)
- Session counter increments on completion
- Offline mode works (service worker caches assets)
- PWA installability criteria

**API Tests (api.spec.ts) - 8 tests:**
- Health endpoint returns ok status
- Quote endpoint returns valid quote structure
- Sessions CRUD operations (create, list, get by id, delete)
- Statistics endpoint aggregates correctly
- Validation errors return 400 status
- Not found returns 404 status

### Lighthouse CI (Quality Metrics)

**Configuration:** `.lighthouserc.cjs`

**Thresholds (all ≥ 80):**
- Performance: 80+
- PWA: 80+
- Accessibility: 80+
- Best Practices: 80+
- SEO: 80+

**Critical Assertions:**
- Service Worker functional
- Installable manifest
- Splash screen configured
- Themed omnibox
- ARIA attributes valid
- Color contrast compliance
- All interactive elements labeled

### Running Tests Locally

1. Build web: `cd apps/web && npm run build`
2. Start API: `cd apps/api && npm start` (background)
3. Start preview: `cd apps/web && npm run preview` (background)
4. Run tests: `cd tests/e2e && npm test`

## Development Notes

### Adding New Timer Features

1. Update `Timer` class in `apps/web/src/timer.js` for state logic
2. Update UI handlers in `apps/web/src/main.js`
3. Update localStorage schema if needed (remember migration)
4. Add E2E tests in `tests/e2e/pwa.spec.ts`

### Adding New API Endpoints

1. Add route handler in `apps/api/src/routes/sessions.js` or new route file
2. Import and mount in `apps/api/src/index.js`
3. Update `apps/web/src/api-client.js` with new method
4. Add E2E tests in `tests/e2e/api.spec.ts`
5. Add validation and error handling

### Modifying PWA Behavior

**Service Worker/Caching:**
- Edit `apps/web/vite.config.js` → `workbox.runtimeCaching`
- Be careful with caching strategies (NetworkFirst, CacheFirst, StaleWhileRevalidate)

**Manifest:**
- Edit `apps/web/vite.config.js` → `manifest` object
- Update icons in `apps/web/public/icons/` if needed

**Base Path (GitHub Pages):**
- `base: '/bootcamp2-chrome-ext-SrMorim/'` in vite.config.js
- Must match repository name for GitHub Pages deployment

### Accessibility

**ARIA Implementation:**
- All interactive elements have `aria-label` or visible label
- Timer display uses `role="timer"` with `aria-live="polite"`
- Status updates use `aria-live` regions
- Mode buttons use `aria-pressed` states
- Tab panels use `role="tabpanel"` with `aria-labelledby`
- Form inputs have `aria-describedby` for hints
- Screen-reader only text uses `.sr-only` CSS class

**Keyboard Navigation:**
- All controls accessible via keyboard
- Focus management for tab switching
- Skip links for main content (if needed)

**Semantic HTML:**
- Proper heading hierarchy (h1 → h2)
- `<main>`, `<nav>`, `<footer>` landmarks
- `role` attributes for custom widgets

### Common Issues

**API not reachable from PWA:**
- Check `VITE_API_URL` environment variable during build
- Check CORS is enabled in API (it is by default)
- In dev: API runs on 3000, web on 8080
- In Docker: API on 3000, web on 80 (mapped to 8080)

**Service worker not updating:**
- Hard refresh (Ctrl+Shift+R)
- Clear application cache in DevTools
- Vite PWA uses `autoUpdate` strategy (should auto-update)

**Tests failing:**
- Ensure API and web preview are running
- Check ports 3000 and 8080 are available
- Verify `E2E_BASE_URL` and `API_URL` env vars
- Check Playwright browsers are installed: `npx playwright install chromium`

**Docker build fails:**
- Ensure npm dependencies are installed in each app
- Check Dockerfile COPY paths are correct
- Verify docker-compose health checks are not timing out

**Lighthouse CI fails:**
- Check web server is running on correct port (8080)
- Verify `.lighthouserc.cjs` configuration
- Review assertions - may need to relax thresholds temporarily
- Common issues: missing aria-labels, color contrast, tap targets

**Unit tests fail:**
- Ensure all dependencies installed: `cd apps/api && npm install`
- Check Node version (requires ≥18 for test runner)
- Integration tests may fail if external API (Quotable) is down - fallback should still pass

**GitHub Pages deployment fails:**
- Check that workflow has correct permissions (pages: write, id-token: write)
- Verify `actions/configure-pages@v5`, `actions/upload-pages-artifact@v3`, and `actions/deploy-pages@v4` are used
- Check that environment `github-pages` is configured in workflow
- Review deployment logs in Actions tab for specific errors
- If migrating from branch-based deployment, old `gh-pages` branch can remain (won't interfere)
- Repository Settings → Pages → Source should show "GitHub Actions" after first successful deploy

### Performance Considerations

- Timer UI updates every second - keep `updateUI()` lightweight
- API sessions endpoint returns all sessions - may need pagination for production
- In-memory sessions storage - use database for persistence
- Service worker caches can grow - monitor cache size in production

## Deployment

### GitHub Pages (Automated)

CI/CD automatically deploys to GitHub Pages on push to main using the official GitHub Actions method:
- Builds web with production API_URL (Render backend)
- Uploads build as Pages artifact via `actions/upload-pages-artifact@v3`
- Deploys using `actions/deploy-pages@v4` (official method)
- Available at: https://srmorim.github.io/bootcamp2-chrome-ext-SrMorim/

**Deployment Method (GitHub Actions Artifact):**
- Uses official GitHub Actions: `configure-pages`, `upload-pages-artifact`, `deploy-pages`
- Automatically configures repository to use "GitHub Actions" as Pages source
- Disables Jekyll processing automatically (no `.nojekyll` needed)
- Environment: `github-pages` with deployment URL tracking
- Concurrency control prevents simultaneous deploys

**Advantages over branch-based deployment:**
- No separate `gh-pages` branch needed
- 100% controlled by CI/CD workflow
- Automatic Jekyll disabling (no configuration needed)
- Better integration with GitHub environments and deployment protection
- Official method recommended by GitHub since 2022

**Repository Settings (Auto-configured):**
After first successful deploy, GitHub automatically sets:
- Source: **GitHub Actions** (auto-detected)
- No manual configuration required

### Docker Deployment

```bash
# Production deployment
docker-compose up -d

# View logs
docker-compose logs -f

# Stop
docker-compose down
```

## Deliveries & Versions

- **v2.0.0** (Final): PWA + Backend + Docker + CI/CD + E2E Tests
- **v1.1.0** (Intermediate): Docker + CI/CD + E2E Tests (Chrome extension)
- **v1.0.0** (Initial): Chrome Extension with Manifest V3

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SrMorim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
