## wapps

> - **Node.js**: v18+ (recommended)

# CLAUDE.md – WonderfulApps Developer Onboarding Guide

## Tech Stack & Versions

### Runtime & Framework
- **Node.js**: v18+ (recommended)
- **Express.js**: ^4.18.2 (actual: 4.18.2+)
- **Runtime**: JavaScript (ES6+)

### Database
- **Primary**: MySQL 5.5.x (Remote Host) / MySQL 8.0 (Local Dev)
- **Driver**: mysql2/promise ^3.6.5
- **Connection Pool**: 50 connections max, queue unlimited
- **Charset**: utf8mb4
- **Date Handling**: DATE fields returned as YYYY-MM-DD strings

### Authentication & Security
- **JWT Library**: jsonwebtoken ^9.0.2
- **Password Hashing**: bcrypt ^5.1.1
- **Authorization**: Bearer token in `Authorization` header
- **Token Expiry**: 8 hours
- **Secret Storage**: `process.env.JWT_SECRET` (environment variable only)

### Validation & Middleware
- **Input Validation**: express-validator ^7.3.1 (actual installed)
- **CORS**: cors ^2.8.5
- **File Upload**: multer ^1.4.5-lts.1
- **HTTP Logging**: morgan ^1.10.0
- **Logging Framework**: winston ^3.11.0

### Frontend & PWA
- **PWA Service Worker**: Registered via `/sw.js`
- **Caching Strategy**: Cache-first for static assets, no-cache for service worker
- **Confetti Animation**: canvas-confetti @1.9.3 (CDN)
- **Input Sanitization**: DOMPurify 2.3.10 (CDN)

### HTTP Client & External APIs
- **HTTP Client**: axios ^1.x (for external API calls)
- **ETF Data APIs**: Tiingo, Finnhub, Polygon (price & reference data via axios)

### Development & Email
- **Environment Variables**: dotenv ^16.3.1
- **Email Service**: Custom `send_email.js` (Resend HTTP-based)
- **Reverse Geocoding**: Nominatim (OSM API proxy via `/routes/geocode.js`)

---

## Core Commands

### Development
```bash
# Start development server (nodemon recommended)
npm start

# Start with explicit port
PORT=8080 npm start

# Run with verbose logging
NODE_ENV=development npm start
```

### Testing
```bash
# Test database connection
node testConnection.js

# Hash a password (one-off utility)
node hash.js
```

### Production
```bash
# Set environment to production
NODE_ENV=production npm start

# Run on non-standard port
PORT=3000 NODE_ENV=production npm start
```

### Database
- **Restore Schema**: Import `wappsDump.sql` into your MySQL instance
- **Connection String**: Built from `.env` variables (`DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_PORT`)

---

## Database Rules & Architecture

### Critical Constraints
1. **Foreign Key Constraints with Cascading Deletes**: All tables with parent references use `ON DELETE CASCADE ON UPDATE CASCADE`.
   - Tables with FKs: ActivitiesT, etfCategoryT, etfSymbolT, InterestEarnedT, TrackUsageT, UserSequenceT, WeightActivitiesT, WeightsT
   - All cascade to UsersT (or intermediate tables)
   - Implication: Deleting a user automatically deletes child records at the database level; no app-level cascade code needed.

2. **MySQL 5.5 Compatibility**:
   - No `JSON` data type (avoid storing JSON columns)
   - No `GENERATED` columns
   - Limited window functions (use application-level grouping)
   - Date strings must be in `YYYY-MM-DD` format

3. **Dual-Key Architecture**: `AUTO_INCREMENT` primary keys + user-scoped secondary keys
   - Global PKs (WeightID, ActivityID, IntErndID, etc.) ensure app-wide uniqueness
   - User-scoped IDs (UserWeightID, UserActivityID, UserIntErndID, etc.) via `UserSequenceT` table provide per-user sequences (1,2,3...)
   - Both are actively used: PKs for joins, user-scoped IDs for user-friendly URLs and API parameters
   - Managed via `dbConnection.js::getNextUserSpecificID(userId, tableName)` — generates and increments UserSequenceT entries
   - FK to UsersT: `ON DELETE CASCADE` removes sequence data when user is deleted

### Key Tables & Relationships

| Table | Purpose | Parent | Child Seq |
|-------|---------|--------|-----------|
| `UsersT` | User accounts | — | UserSequenceT |
| `WeightsT` | Weight entries | UsersT | UserWeightID |
| `ActivitiesT` | Activity definitions | UsersT | UserActivityID |
| `WeightActivitiesT` | Weight-Activity mapping | WeightsT, ActivitiesT | (none) |
| `InterestEarnedT` | Interest contracts | UsersT | UserIntErndID |
| `TrackUsageT` | Page views & time spent | UsersT | (none) |
| `etfCategoryT` | ETF portfolio categories | UsersT | (none) |
| `etfSymbolT` | ETF symbols & metadata | UsersT, etfCategoryT | (none) |
| `UserSequenceT` | ID generation (manual) | UsersT (implicit) | (none) |

### Data Flow Pattern
```
User Input (Frontend)
  → Validation (express-validator)
  → Auth Middleware (JWT check)
  → Route Handler (routes/*.js)
  → Transaction Wrapper (withTransaction)
  → Pool Query (mysql2/promise)
  → Response JSON
```

---

## Security & Authentication

### Mechanism
- **JWT (JSON Web Tokens)** issued upon successful login
- Token contains: `{ userId, username, expiresIn: '8h' }`
- Verified in `middleware/auth.js` before protected routes

### Token Lifecycle
1. **Registration** (`/users/register`): Hash password with bcrypt (10 rounds), generate token
2. **Login** (`/users/login`): Verify credentials, issue token
3. **Protected Routes**: Extract token from `Authorization: Bearer <token>` header
4. **Token Expiry**: 8 hours; client must re-login

### Security Notes
- **Remote Database**: Development only; no production PII storage
- **SSL/TLS**: 
  - Enabled locally (MySQL 8.0)
  - Disabled on remote host (MySQL 5.5 provider limitation)
- **Password Storage**: Never stored as plaintext; always bcrypt (10 rounds)
- **Environment Variables**: All secrets in `.env` file (not in repo)

### CORS Configuration
```javascript
// Allowed Origins
- http://localhost
- http://localhost:8080
- https://wapps.helioho.st
- https://wapps-ypez.onrender.com
```

---

## Coding Patterns & Conventions

### Error Handling
- **Pattern**: `handleDbError(error, res, 'Custom Message')` in `utils.js`
- **Conventions**:
  - Database errors → 500 with error message
  - Duplicate entries → 400 with "Duplicate entry(be)"
  - Validation errors → 400 with specific field error
  - Not found → 404 with "Resource not found(be)"
- **Note**: Error messages end with `(be)` for backend identification

### Validation
- Use `express-validator` chains: `body()`, `param()`, `query()`
- Always call `validationResult(req)` and return early with 400 if errors
- Custom messages use `(be)` suffix convention

### Transaction Pattern
```javascript
await withTransaction(async (connection) => {
  // All queries use 'connection' object, not 'pool'
  const [result] = await connection.query(sql, params);
  // Auto-rollback on error, auto-commit on success
});
```

### Middleware Stack
1. CORS
2. JSON/URL-encoded parsing
3. Multer (file upload)
4. Static file serving (with cache headers)
5. Morgan logging
6. Route handlers
7. Auth middleware (per route)
8. Global error handler

### Naming Conventions
- **Tables**: PascalCase + `T` suffix (e.g., `UsersT`, `WeightsT`)
- **Columns**: PascalCase (e.g., `UserID`, `DateWeight`, `PasswordHash`)
- **Routes**: Kebab-case URLs (e.g., `/users/login`, `/weights/range`)
- **Route Files**: camelCase (e.g., `weightActivities.js`)
- **Error Codes**: All end with `(be)` for backend identification

### Frontend-Backend Contract
- **Request Format**: JSON with `Content-Type: application/json`
- **Response Format**: `{ message: string, data?: object, error?: string }`
- **Auth Header**: `Authorization: Bearer <token>`
- **Date Format**: YYYY-MM-DD (ISO 8601)
- **Numbers**: Floats for decimal values (e.g., weight, interest rate)

### Logging
- Framework: **Winston** (`logger.js`)
- Levels: `info`, `warn`, `error`
- Output: Console + File (`logs/combined.log`, `logs/error.log`)
- Usage: `logger.info('Message')`, `logger.error('Message')`

---

## Environment Configuration

### Required .env Variables
```
# Database
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=<password>
DB_NAME=wonderfulapps
DB_PORT=3306

# Authentication
JWT_SECRET=<random-long-string>

# Email (Resend)
RESEND_API_KEY=<your-api-key>
MAIL_FROM_ADDRESS=<sender-email>
MAIL_FROM_NAME=<sender-name>
MAIL_TO_ADDRESS=<recipient-email>

# ETF External APIs
TIINGO_API_KEY=<tiingo-api-key>
FINNHUB_API_KEY=<finnhub-api-key>
POLYGON_API_KEY=<polygon-api-key>

# Server
PORT=8080
NODE_ENV=development
```

### Local Development Differences
- **MySQL Version**: 8.0 (supports Performance Schema, SSL)
- **SSL**: Enabled in connection
- **Database**: Local instance

### Remote Host Differences
- **MySQL Version**: 5.5.x (legacy provider)
- **SSL**: Disabled (provider limitation)
- **Performance Schema**: Disabled
- **Implication**: Must avoid MySQL 5.5-incompatible features

---

## Important Notes for Contributors

1. **Always use `withTransaction()`** for multi-step operations (INSERT + UPDATE)
2. **FK constraints are in place with CASCADE rules**; rely on database-level cascading deletes for referential integrity
3. **Date handling**: Accept YYYY-MM-DD from frontend; store as DATE; return as YYYY-MM-DD string
4. **User-Specific IDs**: Always use `getNextUserSpecificID(userId, tableName)` to populate User*ID columns (e.g., UserWeightID, UserActivityID); auto-increment PKs are assigned by MySQL
5. **Validation**: Always validate input with express-validator before DB queries
6. **Error Messages**: Append `(be)` to backend-generated error messages
7. **Token Security**: Never log token values; always sanitize in error messages
8. **CORS**: Review allowed origins before deployment
9. **ETF Caching**: `/etf/compare` endpoint caches results per (userId, category) for 60 minutes; use `?nocache=true` during testing
10. **ETF API Keys**: Tiingo, Finnhub, Polygon keys required in `.env`; validate via Polygon in symbol POST request

---

## File Structure Overview

```
root/
├── server.js              # Express app setup, middleware, route mounting
├── dbConnection.js        # MySQL pool, getNextUserSpecificID()
├── logger.js              # Winston logger config
├── utils.js               # handleDbError(), withTransaction()
├── send_email.js          # Email service
├── hash.js                # Password hashing utility (one-off)
├── testConnection.js      # DB connection test
├── debug_etf_compare.js   # ETF /compare endpoint debugging script
├── .env                   # Secrets (not in repo)
├── package.json           # Dependencies
├── routes/
│   ├── users.js           # /users (register, login, logout)
│   ├── weights.js         # /weights (CRUD + range delete)
│   ├── activities.js       # /activities (CRUD)
│   ├── weightActivities.js # /weightActivities (mapping)
│   ├── interestEarned.js   # /interestEarned (CRUD)
│   ├── etf.js             # /etf (category, symbol, compare CRUD + Tiingo/Finnhub proxies)
│   ├── track.js           # /track (analytics)
│   └── geocode.js         # /geocode (reverse geocoding proxy)
├── middleware/
│   ├── auth.js            # JWT verification
│   └── handleValidationErrors.js
├── httpdocs/
│   ├── index.html         # Splash screen
│   ├── home.html          # Main app hub
│   ├── login.html         # Login form
│   ├── register.html      # Registration form
│   ├── Weights.html       # Weight tracking UI
│   ├── Activities.html    # Activity management UI
│   ├── InterestEarned.html # Interest calculator
│   ├── amortization.html  # Loan calculator
│   ├── etf.html           # ETF comparison & analysis hub
│   ├── etfSymbol.html     # Add/manage ETF symbols
│   ├── etfCategory.html   # Manage ETF categories
│   ├── etfCompare.html    # ETF performance comparison table
│   ├── etfActivity.html   # ETF research & activity log
│   ├── etfAPItest.html    # ETF API testing & debugging tool
│   ├── propertyInfo.html  # Property research tools
│   ├── track.html         # Analytics dashboard
│   ├── TownNotice.html    # Geolocation alerts
│   ├── ContactUs.html     # Contact form
│   ├── sw.js              # Service worker (PWA)
│   ├── manifest.json      # PWA manifest
│   └── js/
│       └── confetti.js    # Celebration animation
├── logs/
│   ├── error.log          # Error-only logs
│   └── combined.log       # All logs
└── wappsDump.sql          # Database schema (ground truth)
```

---

## Version History

- **Current**: Node 18+, Express 4.18.2, MySQL 5.5–8.0 compatible
- **Last Updated**: [Current Date]
- **Maintainers**: MPG Jr and team

---

*End of CLAUDE.md*

---
> Source: [sthink1/wapps](https://github.com/sthink1/wapps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
