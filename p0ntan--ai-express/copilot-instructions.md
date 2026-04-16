## ai-express

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Development server**: `npm run dev` - Runs the TypeScript server with hot reload using tsx
- **Production start**: `npm start` - Runs the compiled JavaScript from dist/app.js
- **Linting**: `npx eslint .` - Runs ESLint with TypeScript support (2-space indentation, single quotes, semicolons required)
- **Type checking**: `npx tsc --noEmit` - Validates TypeScript without emitting files

## Architecture Overview

This is an Express.js API server built with TypeScript that provides AI-powered content generation services using OpenAI's GPT-4 Turbo model through the AI SDK.

### Core Structure

- **Entry point**: `src/app.ts` - Express server setup with CORS, middleware, and routing
- **API routes**: Versioned under `/v1` with modular router structure
- **Middleware**: Custom error handling and request parsing (JSON/URL-encoded)
- **Build output**: TypeScript compiles to `dist/` directory

### API Endpoints

The application provides two main services:

1. **Image Alt Text Generation** (`/v1/image/`)
   - `POST /alt-from-url` - Generate alt text from image URL
   - `POST /alt-from-image` - Generate alt text from uploaded image file
   - Supports language specification and custom instructions

2. **SEO Text Improvement** (`/v1/seo/`)
   - `POST /improve-text` - Enhance text for SEO while preserving language and meaning

### Key Dependencies

- **AI SDK**: `ai` package with `@ai-sdk/openai` for OpenAI integration
- **File uploads**: Multer with memory storage for image processing
- **Environment**: dotenv for configuration (requires OPENAI_API_KEY)

### Code Conventions

- ESLint enforces 2-space indentation, single quotes, and semicolons
- TypeScript strict mode enabled
- ES modules with CommonJS compilation target
- Error suppression used for Express error handler typing (`@ts-expect-error`)

### Validation System

The application uses Zod for input validation, similar to ASP.NET Web API model validation:

- **Schemas**: Located in `src/models/` with domain-specific folders (image/, seo/)
- **Validation Middleware**: `validate()` provides route-specific validation
- **Type Safety**: TypeScript types are automatically inferred from Zod schemas
- **Error Handling**: Returns structured validation errors with field-level details

Example usage:
```typescript
import { validate } from '../../validation';
import { SeoImproveTextSchema, UserParamsSchema, SearchQuerySchema } from '../../models';

// Body validation
router.post('/improve-text', validate({
  body: SeoImproveTextSchema
}), async (req, res) => {
  // req.body is now validated and typed
});

// Parameter validation
router.get('/user/:userId', validate({
  params: UserParamsSchema
}), async (req, res) => {
  const { userId } = (req as any).validatedParams || req.params;
});

// Query validation
router.get('/search', validate({
  query: SearchQuerySchema
}), async (req, res) => {
  const { q, limit, offset } = (req as any).validatedQuery || req.query;
});

// File validation
router.post('/upload', upload.single('image'), validate({
  file: {
    fieldName: 'image',
    maxSize: 10 * 1024 * 1024,
    allowedMimeTypes: ['image/jpeg', 'image/png']
  }
}), handler);
```

### Environment Setup

Requires `.env` file with:
- `OPENAI_API_KEY` - OpenAI API key for GPT-4 Turbo access
- `PORT` - Optional, defaults to 3000

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/p0ntan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
