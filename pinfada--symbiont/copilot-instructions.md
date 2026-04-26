## symbiont

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SYMBIONT is a Chrome browser extension that creates evolving digital organisms using AI and WebGL. The project consists of:
- **Frontend Extension**: TypeScript/React with WebGL rendering
- **Backend API**: Express.js server with PostgreSQL/Prisma
- **Multi-layered Architecture**: Content scripts, background workers, popup UI, and social networking

## Development Commands

### Build & Development
```bash
# Full build (extension + backend)
npm run build:all

# Extension only
npm run build

# Development mode with watch
npm run dev

# Backend development
cd backend && npm run dev

# Backend build
cd backend && npm run build
```

### Testing
```bash
# Run all tests
npm test

# Tests with coverage
npm run test:ci

# Watch mode
npm run test:watch

# E2E tests
npm run test:e2e

# Backend tests
cd backend && npm test
```

### Linting & Quality
```bash
# Lint TypeScript
npm run lint

# Fix linting issues
npm run lint:fix

# Check manifest validity
npm run check-manifest
```

### Security & Migration
```bash
# Run security validation
node scripts/validate-security.js

# Migrate Math.random() to SecureRandom
node scripts/migrate-math-random.js

# Test security modules
npm test -- __tests__/security/
```

### Database (Backend)
```bash
cd backend
npm run migrate    # Run Prisma migrations
npm run generate   # Generate Prisma client
npm run studio     # Open Prisma Studio
npm run seed       # Seed database
```

## Architecture Overview

### Core Components
- **OrganismCore** (`src/core/OrganismCore.ts`): Main organism logic with traits, energy, mutations
- **NeuralMesh** (`src/core/NeuralMesh.ts`): Neural network for behavior learning
- **MessageBus** (`src/shared/messaging/MessageBus.ts`): Inter-component communication system
- **SymbiontStorage** (`src/core/storage/SymbiontStorage.ts`): Encrypted local storage

### Security Components
- **SecureRandom** (`src/shared/utils/secureRandom.ts`): Cryptographically secure random number generation
- **SecureLogger** (`src/shared/utils/secureLogger.ts`): GDPR-compliant logging with data sanitization
- **SecurityManager** (`src/background/SecurityManager.ts`): Core security and encryption services

### Extension Architecture
```
Content Script ←→ Background Script ←→ Popup UI
     ↓                    ↓              ↓
Behavior Collection   Neural Core    User Interface
DOM Observation      AI Processing   React Components
User Tracking        WebGL Render    Dashboard/Settings
```

### Message Flow
All components communicate via typed messages through the MessageBus:
- Content scripts collect user behavior → Background for processing
- Background processes data through AI → Popup for visualization
- Social features use P2P messaging between extensions

### Key Directories
- `src/core/`: Core organism and neural network logic
- `src/background/`: Service worker with AI processing
- `src/content/`: DOM observers and behavior collectors  
- `src/popup/`: React UI components and dashboard
- `src/shared/`: Common types, utilities, and messaging
- `src/behavioral/`: Pattern detection and prediction algorithms
- `src/social/`: P2P networking and collective intelligence
- `backend/`: Express API with PostgreSQL database

## TypeScript Configuration

### Path Aliases (Webpack & TSConfig)
```typescript
@/         → src/
@core/     → src/core/
@background/ → src/background/
@content/  → src/content/
@popup/    → src/popup/
@shared/   → src/shared/
@types/    → src/types/
```

### Strict TypeScript Settings
- All strict checks enabled
- No unused locals/parameters
- Exact optional property types
- Force consistent casing

## Testing Strategy

### Coverage Requirements
- Global: 80% (lines, functions, statements), 75% (branches)
- Core modules: 85% coverage minimum
- Utils: 90% coverage minimum

### Test Structure
- Unit tests: `src/**/*.test.ts`
- Integration tests: `tests/integration/`
- E2E tests: `tests/e2e/` (Playwright)
- Mocks: `tests/__mocks__/`

## Chrome Extension Specifics

### Manifest V3 Structure
- Service worker: `src/background/service-worker.ts`
- Content script: `src/content/index.ts`
- Popup: `src/popup/index.tsx`

### WebGL Integration
- Shaders in `src/shaders/` (`.vert`, `.frag` files)
- WebGL orchestrator handles GPU rendering
- Performance monitoring for frame rates

## Backend API

### Technology Stack
- Express.js with TypeScript
- Prisma ORM with PostgreSQL
- WebSocket support via Socket.io
- Redis for caching and sessions
- JWT authentication

### Key Services
- `AuthService`: User authentication and JWT handling
- `DatabaseService`: Prisma client wrapper
- `WebSocketService`: Real-time communication
- `AIService`: ML model integration
- `CacheService`: Redis caching layer

## Security & Privacy

### Cryptographic Security
- **SecureRandom**: All random generation uses `crypto.getRandomValues()` instead of `Math.random()`
- **UUID Generation**: Cryptographically secure UUID v4 with WebCrypto API
- **Secure Logging**: Automatic sanitization of sensitive data (tokens, keys, emails)
- **Production Ready**: FIPS 140-2 compliant random number generation

### Data Protection
- All user data encrypted client-side before storage
- No personal data transmitted without explicit consent
- GDPR compliant data handling with automatic data redaction
- Anonymized social interactions
- Secure logging prevents data leaks in production

### Extension Permissions
- Content script injection for behavior observation
- Storage access for organism persistence
- WebGL access for 3D rendering

### Security Best Practices
- Use `SecureRandom.random()` instead of `Math.random()` for all cryptographic needs
- Use `logger.info()` instead of `console.log()` to prevent data exposure
- Always validate environment variables in production
- Follow principle of least privilege for permissions

## Development Tips

### Common Patterns
- Use `errorHandler` utility for consistent error handling
- Implement `MutationBatcher` for performance-critical operations
- Follow message-driven architecture for component communication
- Use dependency injection in OrganismCore constructor

### Security Patterns
- Import: `import { SecureRandom, logger } from '@/shared/utils';`
- Random numbers: `SecureRandom.random()` instead of `Math.random()`
- Logging: `logger.info('message', data, 'context')` instead of `console.log()`
- UUIDs: `generateSecureUUID()` for cryptographic security
- Environment: Always validate `process.env` variables with fallbacks

### Performance Considerations  
- WebGL operations batched via `WebGLBatcher`
- Neural network processing optimized for real-time use
- Storage operations use hybrid caching strategy
- Message bus includes circuit breaker patterns

### Social Features
- P2P organism sharing via invitation codes
- Collective intelligence through distributed voting
- Mystical events system for community rituals
- Resilient networking with automatic failover

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pinfada) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
