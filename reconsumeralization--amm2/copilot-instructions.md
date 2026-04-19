## amm2

> // Start of Selection

// Start of Selection
---
description: Payload CMS configuration rules and best practices for Modern Men Hair barber
globs:
  - src/collections/**/*.ts
  - src/globals/**/*.ts
  - src/fields/**/*.ts
  - src/hooks/**/*.ts
  - src/endpoints/**/*.ts
  - src/payload.config.ts
  - src/lib/**/*.ts
alwaysApply: true
---

# Payload CMS Development Rules

## Collection Design Patterns
- Structure collections with clear relationships using `relationship` fields
- Implement field-level validation using `validate` functions
- Use `field groups` for organizing related fields visually
- Define access control at both collection and field levels
- Implement custom hooks in `src/hooks/{collection}/{operation}.ts`
- Use `beforeChange`, `afterChange`, `beforeDelete`, `afterDelete` hooks appropriately

## Type Safety Requirements
- Define precise TypeScript types for all collections in `src/payload-types.ts`
- Use `CollectionConfig` type from payload for collection definitions
- Create interfaces for complex field structures
- Export types from central location: `src/types/index.ts`
- Never use `any` type - always provide proper type definitions

## Database Patterns
- Use MongoDB aggregation pipelines for complex queries
- Implement proper indexing for frequently queried fields
- Use transactions for multi-document operations
- Design schemas considering MongoDB document size limits (16MB)
- Implement pagination using `limit` and `skip` parameters

## Security Implementation
- Implement field-level access control using `access` property
- Use environment variables for sensitive configuration
- Sanitize all user inputs using validation functions
- Implement rate limiting for public endpoints
- Use HTTPS for all external communications
- Validate file uploads for type and size restrictions

## API Design
- Create custom endpoints in `src/endpoints/{feature}.ts`
- Use RESTful naming conventions for endpoints
- Implement proper error handling with meaningful messages
- Use consistent response formats across all endpoints
- Document all endpoints with JSDoc comments

## File Organizatio

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/reconsumeralization) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
