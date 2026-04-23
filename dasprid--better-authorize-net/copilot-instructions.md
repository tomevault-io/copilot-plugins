## better-authorize-net

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A type-safe TypeScript client for the Authorize.Net payment gateway API. Uses Zod for runtime validation and `decimal.js` for precision-safe financial calculations. ESM-only, published to npm as `better-authorize-net`.

## Commands

- **Install:** `pnpm install`
- **Build:** `pnpm build` (TypeScript compilation via `tsc -p tsconfig.build.json`)
- **Test:** `pnpm test` (runs with `c8` coverage and `tsx`)
- **Single test:** `npx tsx --test test/path/to/file.test.ts`
- **Lint/fix:** `pnpm check` (Biome lint + format with auto-fix)
- **Format:** `pnpm format` (Biome format only)
- **Regenerate schemas:** `pnpm generate-schemas` (regenerates `src/schemas.ts` from XSD)

## Architecture

### Schema Generation Pipeline

The core of this library is auto-generated Zod schemas derived from Authorize.Net's official XSD schema:

1. `schema-generator/anetapischema.xsd` - The official Authorize.Net XSD schema (source of truth)
2. `schema-generator/generate-schemas.ts` - Parses XSD, builds intermediate representations, topologically sorts by dependency, and emits Zod schemas
3. `schema-generator/intermediate.ts` - Intermediate schema representation classes (RefSchema, SequenceSchema, ChoiceSchema, etc.)
4. `schema-generator/required-on-success-config.ts` - Overrides to make XSD-optional fields required on successful responses
5. `src/schemas.ts` - **Auto-generated, do not edit manually.** Contains all Zod schemas and TypeScript types

### Schema Helpers (`src/schema-helpers.ts`)

Custom Zod codec utilities that handle Authorize.Net's XML-over-JSON quirks:
- `decimalSchema` / `integerSchema` - JSON numbers arrive as strings (via custom reviver) to avoid precision loss, then get decoded to `Decimal` or `number`
- `createArrayWrapSchema` - Authorize.Net's JSON API returns `ArrayOf` types as bare arrays in responses, but expects them wrapped (e.g. `{ "setting": [...] }`) in requests. This codec accepts both formats on parse and always wraps on encode.

### Client (`src/client.ts`)

`AuthorizeNetClient` is the main entry point. It uses a custom JSON reviver to keep all numbers as strings during parsing (preventing floating-point precision loss), then lets Zod schemas handle the string-to-Decimal/number conversion. Each endpoint group is a class that receives a `send` function.

### Endpoint Classes (`src/endpoints/`)

Each file exports an endpoint class (e.g., `PaymentTransactionEndpoints`) that wraps `send()` calls with the correct request/response root keys and Zod schemas. Methods accept typed request objects and return typed response objects.

### Testing Pattern

Tests use a `testEndpoint` helper (`test/test-helpers.ts`) that verifies bidirectional transformation: user input encodes to expected JSON, and server JSON response decodes to expected typed objects. Tests use Node.js built-in test runner with `assert/strict` and `fetch-mock`.

## Conventions

- Commits follow [Conventional Commits](https://www.conventionalcommits.org/) (enforced by commitlint via lefthook)
- Biome handles both linting and formatting (4-space indent, 100 char line width)
- Pre-commit hook runs `biome check --write` on staged files
- Package manager: pnpm (version specified in `packageManager` field)
- Node.js 21+ required (for JSON.parse reviver `context.source` support)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DASPRiD) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
