## stack-auth-astro

> **Package Name**: `astro-stack-auth` (Community Integration)

# Stack Auth Astro Integration - Project Context

## Project Overview

**Package Name**: `astro-stack-auth` (Community Integration)  
**Purpose**: Community-built Stack Auth integration for Astro projects following auth-astro patterns  
**Architecture**: Bridge pattern wrapping Stack Auth's official SDK for Astro compatibility  

## Key Project Invariants

### 1. Architecture Philosophy
- **Bridge Pattern**: Wrap `@stackframe/stack` SDK for seamless Astro integration
- **Zero-config defaults** with environment variable auto-detection
- **Progressive enhancement** from server-only to full React UI
- **Minimal overhead** for performance-conscious applications
- **Community integration** following auth-astro architectural patterns

### 2. Core Dependencies
- `@stackframe/stack` - Stack Auth client SDK
- `@stackframe/stack-ui` - Stack Auth React components  
- **Astro Integration API** - For seamless integration lifecycle
- **Peer dependencies**: Astro 5+, React (for components)

### 3. Package Structure & Entry Points
```
astro-stack-auth/
├── index.js              # Main integration export
├── server.js             # Server-side auth helpers (getUser, requireAuth)
├── client.js             # Client-side auth functions (signIn, signOut)
├── components.js         # React component re-exports from @stackframe/stack-ui
└── middleware.js         # Auth middleware (internal)
```

### 4. Astro Integration Hooks
- **`astro:config:setup`**: Primary integration point
  - Add React renderer for Stack Auth components
  - Inject Stack Auth API route handler via `injectRoute()`
  - Register authentication middleware via `addMiddleware()`
  - Configure environment variables

### 5. Route Injection Pattern
```javascript
injectRoute({
  pattern: `${prefix}/[...stack]`,
  entrypoint: "astro-stack-auth/api/handler", 
  prerender: false
})
```
- Default prefix: `/handler`
- Configurable via options or `STACK_AUTH_PREFIX` env var
- Handles all Stack Auth endpoints (signin, callback, user, session, etc.)

### 6. Middleware Architecture - **IMPLEMENTED ✅**
- **Purpose**: Populate `Astro.locals.user` and `Astro.locals.session`
- **Integration**: Uses `addMiddleware()` with `order: 'pre'`
- **Performance**: Minimal overhead for non-auth requests with intelligent caching
- **Caching**: 5-minute TTL in-memory session cache for optimal performance
- **Stack Auth Integration**: Direct Stack Auth SDK integration with request-based token store
- **Error Handling**: Graceful degradation when Stack Auth is unavailable
- **Security**: Validates session authenticity on every request

### 7. TypeScript Support
```typescript
declare namespace App {
  interface Locals {
    user: import('@stackframe/stack').User | null;
    session: import('@stackframe/stack').Session | null;
  }
}
```

### 8. Environment Variables
**Required:**
- `STACK_PROJECT_ID`
- `STACK_PUBLISHABLE_CLIENT_KEY`
- `STACK_SECRET_SERVER_KEY`

**Optional:**
- `STACK_AUTH_PREFIX` - Custom endpoint prefix

### 9. API Design Patterns

#### Server-Side Helpers (server.js) - **IMPLEMENTED ✅**
```typescript
// Basic authentication functions
getUser(context: APIContext, options?: SecurityValidationOptions): Promise<User | null>
requireAuth(context: APIContext, options?: RequireAuthOptions & SecurityValidationOptions): Promise<User>  
getSession(context: APIContext, options?: SecurityValidationOptions): Promise<Session | null>

// Performance monitoring functions  
getPerformanceStats(): AuthPerformanceStats
getPerformanceSummary(): AuthPerformanceSummary
clearPerformanceData(): void
recordHealthCheck(responseTime: number, success: boolean): void

// Security features
type SecurityValidationOptions = {
  requireSecureTransport?: boolean;
  validateOrigin?: boolean;
  allowedOrigins?: string[];
  requireCSRF?: boolean;
}

// Authentication options
type RequireAuthOptions = {
  signInUrl?: string;
  redirectTo?: string;
}
```

#### Client-Side Functions (client.js) - **IMPLEMENTED ✅**
```typescript
signIn(provider?: string, options?: SignInOptions): Promise<void>
signOut(options?: SignOutOptions): Promise<void>
redirectToSignIn(callbackUrl?: string): void
redirectToSignUp(callbackUrl?: string): void
redirectToAccount(callbackUrl?: string): void
```

#### Component Re-exports (components.js) - **IMPLEMENTED ✅**
```typescript
export { SignIn, SignUp, UserButton, AccountSettings } from '@stackframe/stack-ui'
export { StackProvider } from '@stackframe/stack'
```

### 10. React Component Integration
- **Island Architecture**: Components work with all Astro hydration directives
- **Auto-renderer**: Integration adds React renderer automatically if `addReactRenderer: true`
- **Usage Pattern**: `<SignIn client:load />`, `<UserButton client:visible />`
- **Provider Context**: Components work within Stack Auth's context automatically

### 11. Development Experience Goals
- **5-minute setup**: From `npm install` to working auth
- **Intuitive API**: Follows Astro conventions and patterns
- **Clear errors**: Helpful messages for missing env vars and config issues
- **TypeScript first**: Full type safety throughout

### 12. Sprint Organization
**4 Sprints planned:**
1. **Sprint 001**: Foundation & Setup (Package structure, TypeScript, build system)
2. **Sprint 002**: Core Integration (Middleware, routes, config system)  
3. **Sprint 003**: Server-side Authentication (getUser, requireAuth, protected routes)
4. **Sprint 004**: Client-side & Components (signIn/signOut, React components)

### 13. GitHub Issue Organization
**18 total issues** organized as:
- **10 Feature issues** (#1-#10): User stories for each feature
- **8 Sprint/Task issues** (#11-#18): Implementation tasks

### 14. Testing Strategy
- **Markdown-doctest**: Executable specifications in `/spec/features/*.md`
- **Vitest**: Unit and integration tests
- **Astro Container API**: Testing Astro-specific functionality
- **Playwright**: End-to-end authentication flows

### 15. Inspired by auth-astro Architecture
- **Similar patterns**: Integration wrapping, route injection, middleware
- **Differences**: Targets Stack Auth instead of Auth.js/NextAuth
- **Learnings**: Environment configuration, TypeScript support, developer experience

### 16. Stack Auth Integration Points
- **SDK Wrapping**: Directly uses `@stackframe/stack` SDK methods
- **UI Components**: Re-exports `@stackframe/stack-ui` React components  
- **Session Management**: Leverages Stack Auth's built-in session handling
- **Security**: Uses Stack Auth's CSRF protection and secure defaults

### 17. Key Success Metrics
- **Installation**: Working auth in under 5 minutes
- **Performance**: Minimal bundle impact and runtime overhead
- **Compatibility**: Works with all Astro adapters and rendering modes
- **Type Safety**: Full TypeScript support throughout

### 18. Build & Development
- **Package.json scripts**: test, build, dev tooling configured
- **TypeScript**: Proper configuration with Astro types
- **Vitest**: Modern testing setup with coverage requirements
- **ESLint**: Code quality and consistency

## Sprint 004 Implementation Details (Client-Side & Components) ✅

### Client-Side Authentication Functions
- **`signIn(provider?, options?)`**: Initiates authentication flow with optional provider selection
- **`signOut(options?)`**: Clears session with configurable redirect and cleanup options
- **`redirectToSignIn(callbackUrl?)`**: Direct navigation to Stack Auth sign-in page
- **`redirectToSignUp(callbackUrl?)`**: Direct navigation to Stack Auth sign-up page
- **`redirectToAccount(callbackUrl?)`**: Direct navigation to Stack Auth account management

### React Component Integration
- **`<SignIn />`**: Pre-built sign-in form with provider buttons
- **`<SignUp />`**: Pre-built registration form with validation
- **`<UserButton />`**: User avatar dropdown with account management
- **`<AccountSettings />`**: Full account settings interface
- **`<StackProvider />`**: Context provider for client-side state management

### Client-Side Features
- **Automatic Prefix Discovery**: Client functions auto-detect configured auth prefix
- **Cross-Tab Synchronization**: Authentication state synced across browser tabs
- **Error Boundaries**: Graceful error handling for component failures
- **Hydration Support**: Full compatibility with all Astro hydration directives
- **TypeScript Support**: Complete type definitions for all client functions

### Browser Compatibility
- **Storage Management**: Secure token storage with localStorage/sessionStorage
- **Event Handling**: Browser event listeners for auth state changes
- **Network Resilience**: Retry logic for transient network failures
- **CORS Handling**: Proper CORS configuration for cross-origin requests

## Sprint 003 Implementation Details (Server-Side Authentication) ✅

### Core Server Functions
- **`getUser(context, options?)`**: Retrieves authenticated user from `context.locals`, with optional security validation
- **`getSession(context, options?)`**: Retrieves current session from `context.locals`, with optional security validation
- **`requireAuth(context, options?)`**: Enforces authentication, returns user or throws/redirects based on request type

### Authentication Flow Handling
- **Page Routes**: Redirects unauthenticated users to sign-in page with return URL preservation
- **API Routes**: Returns 401 JSON responses for unauthenticated requests
- **Route Detection**: Automatic detection via path prefix (`/api/`) or `Accept: application/json` header

### Advanced Security Features
- **Rate Limiting**: Configurable rate limits for authentication endpoints
- **Audit Logging**: Comprehensive logging of authentication events and security violations
- **Security Validation**: Optional HTTPS enforcement, origin validation, CSRF protection
- **Performance Monitoring**: Real-time tracking of authentication operation performance
- **Error Recovery**: Graceful handling of Stack Auth service interruptions

### Configuration System
- **Environment Variables**: `STACK_PROJECT_ID`, `STACK_PUBLISHABLE_CLIENT_KEY`, `STACK_SECRET_SERVER_KEY`
- **Custom Prefixes**: Support for custom authentication endpoint prefixes via `STACK_AUTH_PREFIX`
- **Validation**: Comprehensive configuration validation with helpful error messages
- **Development Mode**: Enhanced debugging and logging capabilities

### Middleware Implementation
- **Session Caching**: 5-minute TTL in-memory cache for optimal performance
- **Stack Auth Integration**: Direct SDK integration with automatic token extraction
- **Error Handling**: Graceful degradation when configuration is invalid or service unavailable
- **TypeScript Support**: Full type safety for `Astro.locals.user` and `Astro.locals.session`

### Testing & Validation
- **Unit Tests**: Comprehensive test coverage for all server functions
- **Integration Tests**: End-to-end authentication flow validation
- **Configuration Tests**: Multi-scenario configuration validation
- **Performance Tests**: Authentication operation performance validation
- **Security Tests**: Validation of security measures and error handling

## Current Project Status
- **Phase**: All Sprints Complete ✅
- **Completed Sprints**: 
  - **Sprint 001**: Foundation & Setup ✅
  - **Sprint 002**: Core Integration ✅  
  - **Sprint 003**: Server-side Authentication ✅
  - **Sprint 004**: Client-side & Components ✅
- **Status**: Production-ready with all core features implemented and tested
- **Repository**: Full-featured authentication integration ready for npm publishing

## Key Relationships

**auth-astro → astro-stack-auth**: Architectural pattern inspiration  
**@stackframe/stack → astro-stack-auth**: SDK dependency and wrapping  
**@stackframe/stack-ui → astro-stack-auth**: Component re-exports  
**Astro Integration API → astro-stack-auth**: Core integration mechanics  

This integration bridges Stack Auth's powerful authentication platform with Astro's performance-focused architecture, following proven patterns from the auth-astro community while providing Stack Auth's unique features like multi-tenancy, RBAC, and comprehensive dashboard.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/necanthrope)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/necanthrope)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
