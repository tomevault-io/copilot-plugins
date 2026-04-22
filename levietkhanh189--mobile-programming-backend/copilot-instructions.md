## mobile-programming-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Development with hot reload
npm run dev

# Production build
npm run build
npm start

# Type checking without compilation
npm run type-check
```

## Architecture Overview

This is a TypeScript Express backend following a **layered architecture pattern**:

```
Routes → Controllers → Services → Storage
```

### Key Architectural Patterns

**Layered Separation:**
- **Routes** (`src/routes/`): Define endpoints and mount to Express router
- **Controllers** (`src/controllers/`): Handle HTTP requests/responses, input validation
- **Services** (`src/services/`): Business logic, isolated from HTTP layer
- **Storage** (`src/storage/`): Data persistence layer (currently in-memory)

**Service Layer Design:**
- `auth.service.ts`: JWT generation/verification, password hashing, user registration/login
- `otp.service.ts`: OTP generation, verification, expiry management
- `rate-limit.service.ts`: Time-window based rate limiting for OTP requests
- `user.service.ts`: User data operations
- `email.service.ts`: Email sending (currently console-only, ready for real provider)

**Storage Pattern:**
All storage modules export singleton instances:
```typescript
export const userStorage = new UserStorage();
export const otpStorage = new OTPStorage();
export const rateLimitStorage = new RateLimitStorage();
```

⚠️ **IMPORTANT**: Current storage is **in-memory only**. All data is lost on server restart. This is intentional for development but requires database integration for production.

### Authentication Flow

**JWT-based authentication:**
1. User logs in → receives JWT token (7-day expiry by default)
2. Client sends `Authorization: Bearer <token>` header
3. `authenticateToken` middleware verifies token on protected routes
4. Decoded user info attached to `req.user` for controllers

**Protected routes:**
- Apply `authenticateToken` middleware to routes requiring auth
- User data available via `req.user.userId` and `req.user.email`

### OTP System

**Two registration flows exist:**
1. **Simple registration** (`/auth/register-simple`): Direct registration without email verification
2. **OTP verification** (`/auth/register`): Requires OTP verification before registration

**OTP flow:**
```
1. POST /auth/send-otp → Generates 6-digit code, stores with expiry
2. POST /auth/verify-otp → Validates code, marks as verified
3. POST /auth/register → Checks verification, creates user
```

**Rate limiting:**
- 3 OTP requests per 15 minutes per email (configurable via env vars)
- Implemented via sliding window in `rate-limit.service.ts`

**OTP purposes:**
- `register`: For new user registration
- `forgot-password`: For password reset flow

### Configuration

Environment variables are loaded via `dotenv` and centralized in `src/config/constants.ts`:
- `PORT`: Server port (Railway auto-assigns in production)
- `JWT_SECRET`: **Must be changed from default in production**
- `JWT_EXPIRY`: Token lifetime (e.g., "7d", "24h")
- `OTP_EXPIRY_MINUTES`: OTP validity duration
- `RATE_LIMIT_MAX_REQUESTS`: Max OTP requests per window
- `RATE_LIMIT_WINDOW_MINUTES`: Rate limit time window

### Type Safety

- **Strict TypeScript**: No `any` types in business logic
- Type definitions in `src/types/`:
  - `user.types.ts`: User models and DTOs
  - `auth.types.ts`: JWT payload, auth responses
  - `otp.types.ts`: OTP record structure
  - `express.types.ts`: Extended Express request with user

### Error Handling

- Services throw errors with descriptive messages
- `error.middleware.ts` catches all errors and formats consistent JSON responses
- Controllers should let errors propagate to middleware (don't catch/format manually)

## Production Considerations

**Before deploying to production:**

1. **Database Integration**: Replace in-memory storage with persistent database (MongoDB, PostgreSQL)
2. **Email Service**: Integrate real email provider (SendGrid, Nodemailer with SMTP) in `email.service.ts`
3. **JWT Secret**: Generate strong random secret, never use default value
4. **CORS**: Configure allowed origins instead of allowing all
5. **Rate Limiting**: Consider Redis for distributed rate limiting across multiple instances
6. **Logging**: Add structured logging (Winston, Pino) instead of console.log

**Current limitations:**
- In-memory storage → data loss on restart
- OTP emails only logged to console
- No request logging or monitoring
- CORS allows all origins

## Code Standards

- All files kept under 200 lines (enforced design principle)
- Modular design: each service/controller has single responsibility
- Clear separation between HTTP layer and business logic
- Consistent error messages across all endpoints

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/levietkhanh189) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
