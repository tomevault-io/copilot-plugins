## vikingseventmgmtapi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
- `npm start` - Start production server
- `npm run dev` - Start development server with auto-restart via nodemon
- `npm run build` - No build step needed (returns echo message)

### Code Quality
- `npm run lint` - Run ESLint to check code style and catch errors

### Testing
- `npm test` - Run all tests
- `npm run test:watch` - Run tests in watch mode for development
- `npm run test:coverage` - Run tests with coverage report
- `npm run test:ci` - Run tests in CI mode (no watch, with coverage)

### Release Management & Auto-Deployment Process

**IMPORTANT:** The backend uses **auto-deployment** - code merged to `main` automatically deploys to production on Railway. This requires careful version management and testing.

#### **Auto-Deployment Workflow:**

**Standard Process:**
```bash
# 1. Feature Development (in PR)
git checkout -b feature/my-feature
# ... develop and test ...
npm run lint && npm test && npm run build  # Verify before PR

# 2. Version Management (BEFORE merge)
# Update version in PR if this will be a new release
npm run version:patch  # Updates package.json only (--no-git-tag-version)
git add package.json package-lock.json
git commit -m "chore: bump version to v1.x.x for production deployment"

# 3. PR Review & Merge
# Create PR → Review → Merge to main
# ⚠️ Auto-deployment triggers immediately after merge

# 4. Post-Merge Release Finalization (LOCAL SYNC)
git checkout main && git pull origin main  # Sync with merged changes
git tag -a v1.x.x -m "Release v1.x.x: Description of changes"
git push origin v1.x.x
npm run release:finalize  # Finalize Sentry release
```

#### **Production Issue Resolution Workflow:**

**When fixing critical production errors:**

```bash
# 1. Create Hotfix Branch
git checkout -b feature/hotfix-critical-issue

# 2. Implement Fix with Enhanced Logging
# Add structured logging and error context
# Include Sentry integration for monitoring

# 3. Update Version BEFORE Merge (Critical!)
npm run version:patch
git add package.json package-lock.json  
git commit -m "chore: bump to v1.x.x for critical production fix"

# 4. PR with Detailed Context
# Include Sentry issue IDs and error descriptions
# Reference production monitoring and logs

# 5. Post-Merge: Align Local & Create Release Tag
git checkout main && git pull origin main
git tag -a v1.x.x -m "Release v1.x.x: Critical fixes for [issue description]"
git push origin v1.x.x
npm run release:finalize
```

#### **Release Commands:**

**Development Releases:**
```bash
./scripts/release.sh patch  # Automated version bump + Sentry integration
```

**Production Hotfixes:**
```bash
# In PR branch BEFORE merge:
npm run version:patch
git add package.json package-lock.json
git commit -m "chore: bump version to v1.x.x"

# After merge and sync:
git tag -a v1.x.x -m "Release message"  
git push origin v1.x.x
npm run release:finalize
```

**Manual Release Process (Fallback):**
```bash
npm run release:patch  # Test + bump patch version
npm run release:minor  # Test + bump minor version  
npm run release:major  # Test + bump major version
```

#### **Sentry Integration:**

**Release Management:**
```bash
npm run release:create    # Create new Sentry release
npm run release:finalize  # Finalize release for deployment tracking
npm run release:deploy    # Mark deployment with commit info
```

**Monitoring & Alerting:**
- Structured logging captures all API requests and errors
- Rate limit monitoring with OSM API integration  
- OAuth flow monitoring and error tracking
- Performance profiling in production environments

**Post-Deployment Monitoring:**
- Monitor Sentry for error patterns and rate limit issues
- Check Railway deployment logs for startup errors
- Verify API endpoints are responding correctly
- Monitor OAuth callback functionality

#### **Version Management:**

**Current Scripts:**
- `npm run version:patch` - Bump patch version (no git operations)
- `npm run version:minor` - Bump minor version (no git operations)
- `npm run version:major` - Bump major version (no git operations)
- `npm run build:release` - Build + Sentry integration (no-op for Node.js)
- `npm run release:patch` - Run tests, then bump patch version
- `npm run release:minor` - Run tests, then bump minor version
- `npm run release:major` - Run tests, then bump major version

**Auto-Deploy Considerations:**
- Always bump version BEFORE merging to main
- Test thoroughly including all 53 test cases
- Use descriptive commit messages for deployment tracking
- Monitor Railway logs after deployment

**Critical Production Issues:**
- Version bump is REQUIRED (distinguishes from problematic releases)
- Include enhanced Sentry logging and error context
- Reference specific error patterns or monitoring alerts
- Monitor deployment success via Railway and Sentry integration

Test configuration uses Jest with Node.js environment, 15-second timeout, and single worker to prevent port conflicts.

## Architecture Overview

This is a Node.js Express backend that serves as an OAuth proxy for Online Scout Manager (OSM) API integration.

### Core Components

**Main Server** (`server.js`):
- Express app with CORS configuration for multiple frontend domains
- OAuth callback handling with dynamic frontend URL detection based on state parameter
- Sentry integration for error monitoring
- Rate limiting middleware applied to all routes
- **Refactored with server utility helpers** - 11% code reduction through pattern elimination

**Controllers**:
- `controllers/auth.js` - OAuth token management, logout functionality
- `controllers/osm.js` - **Heavily refactored** OSM API proxy endpoints (86% code reduction)
- `controllers/osm-legacy.js` - Legacy functions with complex business logic (new)

**Middleware**:
- `middleware/rateLimiting.js` - Dual-layer rate limiting (backend + OSM API tracking)

**Configuration**:
- `config/sentry.js` - Sentry error monitoring and structured logging setup

**Utility Modules** (NEW):
- `utils/osmApiHandler.js` - Generic OSM API request handler with built-in validation
- `utils/validators.js` - Reusable validation functions
- `utils/responseHelpers.js` - Standardized response processing
- `utils/osmEndpointFactories.js` - Pre-configured endpoint handlers
- `utils/serverHelpers.js` - Server utility functions to reduce redundancy

### Rate Limiting Architecture

The application implements a sophisticated dual-layer rate limiting system:

1. **Backend Rate Limiting**: 100 requests per minute per user/IP
2. **OSM API Rate Limiting**: Tracks and respects OSM's rate limits per user session

Rate limit information is automatically included in all API responses under `_rateLimitInfo` field.

### OAuth Flow

OAuth authentication supports dynamic frontend URL detection with multiple fallback mechanisms:

**Frontend URL Detection Priority:**
1. `frontend_url` query parameter (highest priority)
2. Embedded URL in state parameter: `state=prod&frontend_url=https://pr-123-vikings-eventmgmt.onrender.com`
3. Referer header detection for `.onrender.com` domains
4. Legacy state-based detection:
   - `state=dev` or `state=development` → `https://localhost:3000`
   - `state=prod` or `state=production` → production frontend
5. Default: `https://vikings-eventmgmt.onrender.com`

This supports PR preview URLs and dynamic environments while maintaining backward compatibility.

Tokens are stored in-memory (Map) with expiration tracking.

### OAuth Redirect URI Troubleshooting

**Critical Requirement**: The `redirect_uri` parameter must match exactly between:
1. Initial OAuth authorization request (generated by frontend)
2. Token exchange request (sent by backend)

**Common Issues:**
- **Environment Variable Mismatch**: Ensure `BACKEND_URL` on deployed backend matches the actual deployment URL
- **Frontend/Backend URL Mismatch**: Frontend's `VITE_API_URL` must match backend's `BACKEND_URL`
- **Copy-paste Errors**: When deploying to new platforms, verify environment variables are updated to match the new deployment URLs

**Debugging Steps:**
1. Check `/oauth/debug` endpoint on backend to verify `backendUrl` and `authUrl` values
2. Verify frontend's `VITE_API_URL` matches backend's `BACKEND_URL`
3. Test OAuth flow locally first to isolate environment vs code issues
4. Compare browser network requests to identify exact redirect URI mismatches

**Example Valid Configuration:**
```env
# Backend (.env)
BACKEND_URL=https://vikingeventmgmtapi-production.up.railway.app

# Frontend (.env)
VITE_API_URL=https://vikingeventmgmtapi-production.up.railway.app
```

### API Endpoints Structure

**Authentication**: `/token`, `/logout`, `/oauth/callback`, `/oauth/debug`
**OSM Proxy**: All use Authorization header with Bearer token
- GET endpoints: `/get-terms`, `/get-section-config`, `/get-user-roles`, `/get-events`, `/get-flexi-records`, `/get-single-flexi-record`, `/get-flexi-structure`, `/get-startup-data`
- POST endpoints: `/update-flexi-record` (with enhanced validation and Sentry logging), `/get-members-grid`
**Utility**: `/rate-limit-status`

### Refactored Endpoint Implementation

**Before Refactoring** (eliminated):
- Each endpoint: 40-80 lines of identical boilerplate code
- Repeated authorization, validation, error handling, and response processing
- ~800 lines of redundant code across 15+ endpoints

**After Refactoring** (current):
- Simple endpoints: 1-2 lines using factory pattern
- Complex endpoints: Custom logic only with shared utilities
- Centralized validation, error handling, and response processing
- 86% reduction in controller code (1,140 → 160 lines)

### Environment Variables Required

```env
OAUTH_CLIENT_ID=your_osm_client_id
OAUTH_CLIENT_SECRET=your_osm_client_secret
NODE_ENV=development|production|test
PORT=3000
SENTRY_DSN=optional_sentry_dsn
BACKEND_URL=backend_base_url
```

### Testing Approach

Tests are located in `__tests__/` directory:
- `server.test.js` - Comprehensive unit tests (53 tests covering all endpoints)
- `integration.test.js` - Integration tests

**Test Coverage (Added by coderabbit)**:
- Rate limiting functionality and middleware
- API endpoint validation and responses  
- CORS configuration across multiple domains
- OAuth flow and callback handling
- Frontend URL detection with security validation
- Health check and admin endpoints
- Documentation serving (Swagger UI)
- Error handling for various scenarios
- Performance and security edge cases

**Recent Test Fixes for CI/CD**:
- Enhanced frontend URL validation with protocol/hostname security checks
- Fixed backend documentation endpoint compatibility (accepts 200/301 status codes)
- Improved Sentry test endpoint default behavior
- Enhanced global error handler to preserve original HTTP status codes
- Added URL length validation for DoS protection (max 1000 chars)

Jest configuration prevents port conflicts with single worker and handles async cleanup properly.

### Sentry Structured Logging

The application implements comprehensive structured logging using Sentry:

**Configuration Features:**
- Structured logging enabled with `_experiments: { enableLogs: true }`
- Console logging integration automatically captures console.log/error/warn
- Error monitoring with OSM-specific context and rate limit information
- Performance profiling in production environments

**Logging Patterns:**
- Import: `const Sentry = require('../config/sentry'); const { logger } = Sentry;`
- Structured context: All logs include `endpoint`, `sessionId`, and relevant parameters
- Log levels: `logger.info()`, `logger.error()`, `logger.warn()`, `logger.debug()`
- Template literals: Use `logger.fmt` for dynamic values in log messages

**Enhanced updateFlexiRecord Logging:**
- Request validation with parameter logging
- OSM API request/response tracking
- Rate limit monitoring and warnings
- Error context with full stack traces
- Success confirmation with operation details

### Key Implementation Notes

- All OSM proxy endpoints extract access tokens from Authorization headers
- Rate limiting headers are automatically added to responses
- Error responses include rate limit information when applicable
- OSM API responses are wrapped with rate limit info before sending to frontend
- Enhanced validation for flexi record updates with field ID format checking
- Comprehensive Sentry logging for debugging and monitoring
- Graceful shutdown handling with SIGTERM

**Security Enhancements**:
- Frontend URL validation with whitelist approach for security
- Protocol validation (HTTP only allowed for localhost/127.0.0.1)
- URL length limits to prevent DoS attacks (max 1000 characters)
- Enhanced error handling preserving original HTTP status codes
- Comprehensive input validation across all endpoints

**CI/CD Integration**:
- All 53 tests passing for automated deployment pipeline
- GitHub Actions compatible test configuration
- Comprehensive edge case coverage for production reliability

### Refactoring Implementation Details

**Utility Functions**:
- `createOSMApiHandler()` - Generic handler with built-in authorization, validation, and error handling
- `osmEndpoints.*()` - Pre-configured endpoint factories for simple GET/POST operations
- `validateRequiredParams()` - Centralized parameter validation with consistent error messages
- `sendOSMResponse()` - Standardized response processing with rate limit information
- `createEndpointLogger()` - Structured logging with consistent format and context

**Factory Pattern Benefits**:
- New endpoints can be created in minutes vs hours
- Consistent error handling across all endpoints
- Single source of truth for common operations
- Reduced maintenance burden and bug risk
- Improved code readability and developer experience

**Architecture Improvements**:
- **DRY Principle**: Eliminated code duplication through shared utilities
- **Separation of Concerns**: Business logic separated from boilerplate code
- **Testability**: Common functionality easier to test in isolation
- **Maintainability**: Changes to common behavior require updates in one place only
- **Consistency**: Standardized patterns across all endpoints

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Walton-Viking-Scouts)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Walton-Viking-Scouts)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
