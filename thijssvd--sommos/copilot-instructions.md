## sommos

> SommOS Architecture and Code Organization Patterns


# SommOS Architecture Patterns

## Modular Structure
- API routes belong in `/backend/api/`
- Business logic belongs in `/backend/core/`
- Database operations belong in `/backend/database/`
- Frontend components belong in `/frontend/js/`
- Schemas and validation belong in `/backend/schemas/`

## Error Response Format
Always use the established error response format:
```javascript
{
  success: boolean,
  error?: string,
  data?: any,
  meta?: object
}
```

## Async Patterns
- Use async/await instead of callbacks or .then() chains
- Always include proper error handling with try-catch blocks
- Implement timeout handling for external API calls (30s for AI, 10s for others)

## Naming Conventions
- Use snake_case for database fields and API parameters
- Use camelCase for JavaScript variables and functions
- Use PascalCase for class names
- Use UPPER_SNAKE_CASE for constants

## Class Structure
Follow the established class patterns:
```javascript
class SommOSAPI {
    constructor() {
        this.baseURL = this.determineBaseURL();
        this.timeout = 30000;
    }
}
```

## Offline-First Architecture
- Maintain PWA capabilities with service worker patterns
- Use IndexedDB for local storage
- Implement graceful degradation when offline
- Cache static assets with proper versioning

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Thijssvd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
