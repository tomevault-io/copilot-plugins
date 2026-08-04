## product-catalog-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Development server with hot reload
pnpm dev

# Build for production (TypeScript compilation + alias resolution)
pnpm build

# Start production server
pnpm serve
```

## Architecture Overview

This is a TypeScript Express.js API that acts as a proxy/wrapper around the DummyJSON API (https://dummyjson.com), providing a simplified product catalog interface.

### Key Architectural Patterns

**Mock API Proxy Pattern**: The application fetches data from DummyJSON and transforms it into a standardized format. External API responses are mapped to internal `Product` interface with 7 core fields (id, name, sku, description, category, image, brand).

**Centralized Error Handling**: Error responses are generated through function factories in `/src/error/`:
- `PRODUCTS_RESPONSE_ERROR` - Handles /products endpoint validation
- `SEARCH_PRODUCTS_RESPONSE_ERROR` - Handles /products/search endpoint validation

Each error function returns descriptive messages with the invalid parameter value included.

**Dual Type System**:
- External types (e.g., `DummyProductsResponse`) mirror the DummyJSON API structure
- Internal types (e.g., `Product`, `ProductsResponse`) provide simplified, consistent interfaces
- Request types define query parameter validation schemas

**Validation-First Routing**: All Express routes validate required parameters first and return early with 400 status + descriptive errors before processing requests.

### Project Structure

```
src/
├── error/           # Error response generators
├── mock-api/        # DummyJSON API integration layer
├── types/           # TypeScript interfaces (internal + external)
├── lib/             # Shared constants
└── index.ts         # Express server + route definitions
```

### TypeScript Configuration

- Uses `@/` path aliases for clean imports
- Development: `tsconfig-paths` for runtime alias resolution
- Production: `tsc-alias` for build-time alias compilation
- Strict TypeScript settings with source maps enabled

### API Endpoints

**GET /products**: Paginated product listing with sorting
- Required: `limit`, `page` 
- Optional: `sortBy` (default: 'sku'), `sortByDirection` (default: 'desc')

**GET /products/search**: Product search functionality
- Required: `q` (query string), `limit`, `page`

Both endpoints return consistent `ProductsResponse` format with `products` array, `total`, `skip`, and `limit` fields.

### Error Response Format

All validation errors return:
```json
{
  "products": [],
  "error": "Descriptive error message with parameter value"
}
```

### Dependencies

- **Runtime**: Express.js, CORS
- **Development**: TypeScript, ts-node, path alias tools
- **Package Manager**: pnpm (v10.13.1)
- **External API**: DummyJSON (https://dummyjson.com) for product data

---
> Source: [abubakir1997/product-catalog-api](https://github.com/abubakir1997/product-catalog-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
