## happygotchadays

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**HappyGotchaDays.com** - A Progressive Web App for pet owners to celebrate and share their dogs' and cats' adoption anniversaries ("gotcha days"). Built on Cloudflare Workers edge platform.

**Current State:** Fully deployed and operational at https://happygotchadays.bill-burkey.workers.dev

**Critical Context:** This platform is **exclusively for pet (dog/cat) adoption celebrations**. The term "gotcha day" is controversial in child adoption contexts. Never conflate pet and human adoption.

## Technology Stack

- **Runtime:** Cloudflare Workers (edge compute)
- **Framework:** Hono (lightweight web framework)
- **Database:** Cloudflare D1 (SQLite at edge)
- **Storage:** Cloudflare R2 (object storage for photos)
- **Cache:** Cloudflare KV (sessions and caching)
- **AI:** Cloudflare Workers AI
- **Auth:** Custom JWT with KV sessions
- **Frontend:** Vanilla JavaScript/HTML/CSS served from Worker
- **PWA:** Service Worker, manifest, offline support

## Architecture

### Request Flow
1. Request → Cloudflare Worker (Hono app)
2. Logger & CORS middleware
3. Route matching (`/api/*` or static files)
4. Authentication middleware (where required)
5. Route handler (database/R2/AI operations)
6. Response

### Static File Serving
Frontend assets are embedded as JavaScript strings and served via the Worker itself (no separate static hosting). See `src/middleware/static.js` and `src/frontend/*`.

### Database Design
All tables use `TEXT` for IDs (UUIDs) and `INTEGER` for timestamps (Unix epoch). Key relationships:
- Users → Pets (one-to-many)
- Pets → Photos (one-to-many)
- Users → Posts → Likes/Comments (many-to-many)
- Users ↔ Users (follows, many-to-many)

## Development Commands

```bash
# Local development
npm run dev                    # Start local Worker (localhost:8787)
curl http://localhost:8787/health  # Quick health check

# Database operations
npm run db:migrate:local       # Apply migrations locally
npm run db:migrate             # Apply migrations to remote D1
wrangler d1 execute gotchadays-db --local --command="SELECT * FROM users LIMIT 5"  # Query local DB
wrangler d1 execute gotchadays-db --remote --command="SELECT * FROM users LIMIT 5" # Query remote DB

# Testing
npm test                       # Run all tests with Vitest
npm test -- tests/unit/auth.test.js  # Run specific test file
npm run test:coverage          # Run tests with coverage

# Deployment
npm run deploy                 # Deploy to production
npm run deploy:preview         # Deploy to preview environment

# Secrets and configuration
wrangler secret list           # List configured secrets
wrangler secret put JWT_SECRET # Set JWT_SECRET (prompted for value)
wrangler kv:namespace list     # List KV namespaces
wrangler r2 bucket list        # List R2 buckets
```

## Key Files & Locations

### Entry Point
- `src/index.js` - Main Worker, sets up Hono app with all routes

### Routes (6 API modules)
- `src/routes/auth.js` - Register, login, logout, verify
- `src/routes/pets.js` - Pet profile CRUD
- `src/routes/photos.js` - Photo upload/download via R2
- `src/routes/social.js` - Posts, likes, comments, follows
- `src/routes/search.js` - Search pets/users, discover, upcoming gotcha days
- `src/routes/reminders.js` - Gotcha day reminder management (email notifications)

### Utilities
- `src/utils/jwt.js` - JWT signing/verification using Web Crypto API
- `src/utils/password.js` - Password hashing with SHA-256
- `src/utils/id.js` - UUID generation and timestamps

### Middleware
- `src/middleware/auth.js` - JWT authentication middleware
  - `authMiddleware` - Requires valid JWT, attaches user to context as `c.get('user')`
  - `optionalAuth` - Checks for JWT but doesn't require it (for public/private content)
- `src/middleware/static.js` - Serves frontend from embedded strings

### AI Integration
- `src/ai/image-analysis.js` - Breed detection, descriptions, moderation, tagging
- `src/ai/content-generation.js` - Story prompts, celebration messages, hashtags

### Frontend (embedded as JS strings)
- `src/frontend/index.html.js` - Main HTML structure
- `src/frontend/styles/main.css.js` - Complete CSS including modals
- `src/frontend/scripts/app.js.js` - Client-side JavaScript
- `src/frontend/manifest.json.js` - PWA manifest
- `src/frontend/sw.js.js` - Service Worker

## Environment Bindings

Accessed via `c.env` in Hono context:

- `DB` - D1 database (`gotchadays-db`)
- `PHOTOS` - R2 bucket (`gotchadays-photos`)
- `SESSIONS` - KV namespace for user sessions
- `CACHE` - KV namespace for general caching
- `AI` - Workers AI binding
- `JWT_SECRET` - Secret for signing tokens
- `ANALYTICS` - Analytics Engine (optional)

## API Patterns

### Route Structure
All route files export a Hono router instance:
```javascript
import { Hono } from 'hono';
export const myRoutes = new Hono();
myRoutes.get('/endpoint', async (c) => { /* handler */ });
```
Register in `src/index.js` with: `app.route('/api/prefix', myRoutes)`

### Authentication
Routes using `authMiddleware` require JWT and attach user to context:
```javascript
import { authMiddleware } from '../middleware/auth.js';
myRoutes.get('/protected', authMiddleware, async (c) => {
  const user = c.get('user'); // { id: string, email: string }
  // ...
});
```
Use `optionalAuth` for endpoints that work for both authenticated and guest users.

### Database Queries (D1)
All tables use `TEXT` for IDs and `INTEGER` for timestamps. D1 uses prepared statements:
```javascript
// Single result
const user = await c.env.DB.prepare(
  'SELECT * FROM users WHERE id = ?'
).bind(userId).first();

// Multiple results
const pets = await c.env.DB.prepare(
  'SELECT * FROM pets WHERE user_id = ?'
).bind(userId).all();

// Insert/Update/Delete
const result = await c.env.DB.prepare(
  'INSERT INTO pets (id, user_id, name, species, gotcha_date, created_at) VALUES (?, ?, ?, ?, ?, ?)'
).bind(id, userId, name, species, gotchaDate, timestamp).run();
```

### R2 Storage
```javascript
// Upload with metadata
await c.env.PHOTOS.put(key, fileStream, {
  httpMetadata: { contentType: 'image/jpeg' }
});

// Retrieve
const object = await c.env.PHOTOS.get(key);
const arrayBuffer = await object.arrayBuffer();

// Delete
await c.env.PHOTOS.delete(key);
```

### Workers AI
```javascript
// Image classification
const result = await c.env.AI.run('@cf/microsoft/resnet-50', { image: arrayBuffer });

// Vision model
const result = await c.env.AI.run('@cf/llava-hf/llava-1.5-7b-hf', {
  image: arrayBuffer,
  prompt: "Describe this pet"
});

// Text generation
const result = await c.env.AI.run('@cf/meta/llama-3.1-8b-instruct', {
  messages: [{ role: "user", content: "Generate a story..." }]
});
```

## Important Patterns

### Frontend Embedded as Strings
The frontend is **not** separate files - it's embedded as exported strings in `.js` files. When editing frontend:
1. Edit `src/frontend/*.js` files
2. Modify the template literal content
3. Deploy to update

### Modal Pattern
Modals are dynamically created by injecting HTML into `#modalContainer`. Close by clicking backdrop or calling `closeModal()`. Requires proper CSS (already added).

### Adding New API Endpoints
When adding new endpoints:
1. Add handler to appropriate route file in `src/routes/` (or create new route file)
2. If new route file, register it in `src/index.js`: `app.route('/api/prefix', newRoutes)`
3. Add migration if database changes needed in `migrations/`
4. Run `npm run db:migrate:local` to apply locally
5. Test with `npm run dev`
6. Run `npm run deploy` to push changes (frontend is bundled with Worker)

### Error from Missing Endpoints
If frontend calls an API endpoint that doesn't exist (like `/api/auth/verify` was initially), add it to the appropriate route file and redeploy.

## Common Issues

### "404 Not found" on API calls
- Check route is registered in `src/index.js`
- Verify route file exports with correct Hono pattern
- Check HTTP method matches (GET/POST/PUT/DELETE)

### Frontend not updating
- Frontend is cached in the Worker bundle
- Must redeploy with `npm run deploy` to see changes
- Service Worker also caches - hard refresh in browser (Cmd+Shift+R)

### Database errors
- Ensure migrations ran: `npm run db:migrate`
- Check binding names in `wrangler.toml` match code
- Verify data types match schema (TEXT for IDs, INTEGER for timestamps)

### Authentication not working
- Verify `JWT_SECRET` is set: `wrangler secret list`
- Check token is sent in `Authorization: Bearer <token>` header
- Token expires after 1 hour (3600 seconds)

## Workers AI Models

### Image Analysis
- `@cf/microsoft/resnet-50` - Image classification, breed detection
- `@cf/llava-hf/llava-1.5-7b-hf` - Vision model for descriptions and moderation

### Content Generation
- `@cf/meta/llama-3.1-8b-instruct` - Text generation for stories, prompts, hashtags

## Deployment Info

**Live URL:** https://happygotchadays.bill-burkey.workers.dev
**GitHub:** https://github.com/abandini/happygotchadays
**Cloudflare Account:** bill.burkey@ememetics.com
**Worker Name:** `happygotchadays`

### GitHub Actions
Workflow at `.github/workflows/deploy.yml` auto-deploys on push to `main` (requires `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID` secrets).

## Testing

### Unit Tests
- Located in `tests/unit/`
- Run specific test: `npm test -- tests/unit/auth.test.js`
- Current tests: `auth.test.js`, `reminders.test.js`
- Use Vitest for test framework
- Mock Cloudflare bindings (DB, KV, R2, AI) in tests

### Integration/E2E Tests
- Located in `tests/integration/` and `tests/e2e/`
- Test against local Wrangler dev server
- Browser console test script in `BROWSER_TEST.md` for full API workflow

### Manual Testing
1. Visit https://happygotchadays.bill-burkey.workers.dev
2. Click "Sign Up" - modal should appear
3. Fill form and submit - should create account
4. Navigation should update to show authenticated state

## Performance Notes

- Worker response time target: <100ms (excluding AI)
- D1 queries target: <50ms
- Images should be <500KB
- Initial page load target: <2s
- Use KV for caching frequently accessed data

## Security

- Passwords hashed with SHA-256 (Web Crypto API)
- JWT tokens expire after 1 hour
- Sessions stored in KV with TTL
- All inputs validated before database operations
- Parameterized queries prevent SQL injection
- CORS configured for specific origins
- Rate limiting should be implemented (not yet done)

## Coding Conventions

### Style Guidelines
- **Module System:** ES modules (`"type": "module"` in package.json)
- **Indentation:** 2 spaces (no tabs)
- **Quotes:** Single quotes for strings
- **Naming:** `camelCase` for variables/functions, `PascalCase` for classes
- **Async:** All route handlers should be async
- **Comments:** Use block comments above handlers for non-trivial flows; avoid inline noise

### Route File Pattern
```javascript
import { Hono } from 'hono';
import { authMiddleware } from '../middleware/auth.js';

export const myRoutes = new Hono();

// GET endpoint with auth
myRoutes.get('/resource', authMiddleware, async (c) => {
  const user = c.get('user');
  // Handler logic
  return c.json({ data: result });
});

// POST endpoint
myRoutes.post('/resource', authMiddleware, async (c) => {
  const body = await c.req.json();
  // Handler logic
  return c.json({ success: true }, 201);
});
```

### Frontend Asset Pattern
Frontend files in `src/frontend/` are JavaScript modules that export template literal strings:
```javascript
// src/frontend/example.html.js
export const exampleHTML = `
<!DOCTYPE html>
<html>
  <!-- HTML content -->
</html>
`;
```

## Market Context

- 4.2 million annual pet adoptions in US
- 81% of pet parents celebrate gotcha days
- Target: Millennials (33% of pet owners), middle to upper-middle income
- This is a joyful celebration platform for rescue pets

## References

- [Cloudflare Workers](https://developers.cloudflare.com/workers/)
- [Hono Framework](https://hono.dev/)
- [Cloudflare D1](https://developers.cloudflare.com/d1/)
- [Cloudflare R2](https://developers.cloudflare.com/r2/)
- [Cloudflare Workers AI](https://developers.cloudflare.com/workers-ai/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abandini)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/abandini)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
