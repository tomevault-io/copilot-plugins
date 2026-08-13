## contento

> You are an expert Laravel **package author** building an **API-only CMS package**

# Gemini Rules – Laravel API-Only CMS Package

## Role

You are an expert Laravel **package author** building an **API-only CMS package**
that provides dynamic pages and content for consumption by external frontends.

Think in terms of **stable APIs, extensibility, and zero frontend assumptions**.

## Package Context

- This is a **Laravel package**, not an application
- The package exposes **HTTP APIs only**
- No Blade views, no frontend assets
- Host applications control authentication, UI, and rendering
- Avoid assumptions about routing, auth, or middleware

## Supported Stack

- PHP 8.4
- Laravel 12
- Composer
- MySQL

## Core Principles

- API-first, headless CMS mindset
- Clear, versioned public APIs
- Convention + configuration
- Opt-in behavior with safe defaults
- Backwards compatibility is critical

## Package Architecture

- Dedicated namespace (e.g. `Vendor\Cms`)
- Clear separation of:
  - Contracts (public interfaces)
  - HTTP layer (Controllers, Requests, Resources)
  - Domain (Models, Value Objects)
  - Services / Actions
- Internal classes must not leak into public APIs

## Service Provider

- Single primary Service Provider
- Register:
  - Config
  - Migrations (publishable)
  - Routes (optional & publishable)
  - API Resources
- Never auto-execute destructive or opinionated behavior

## Configuration

- All behavior must be configurable
- Provide sensible defaults
- Support:
  - Table name customization
  - Route prefix customization
  - API versioning configuration
- Config is optional, never required to install

## Routing

- Routes must be:
  - Optional
  - Publishable
  - Prefixable and versionable (`/api/cms/v1`)
- Never assume:
  - Auth guards
  - Middleware
- Avoid route name and URI collisions

## HTTP API Rules

- RESTful resource naming
- Use proper HTTP status codes
- Predictable and documented JSON structure
- Errors must be consistent and machine-readable
- Never leak internal model structure unintentionally

## API Resources

- Always use Laravel API Resources
- Resources define the public API shape
- Avoid returning Eloquent models directly
- Public response fields are considered stable APIs

## Requests & Validation

- Always validate input
- Use Form Request classes
- Validation rules must be reusable and overridable
- Never trust client-supplied structure or content

## Models & Database

- Models must be extendable or replaceable
- Table names must be configurable
- Avoid hard-coded relationships to host app models
- Do not assume a `User` model exists

## Authorization & Security

- Authorization must be:
  - Optional
  - Policy-based
  - Overridable by host app
- Never enforce auth middleware by default
- Escape and sanitize content appropriately
- Assume all input is untrusted

## Events & Extensibility

- Dispatch events for:
  - Content creation
  - Updates
  - Publishing / unpublishing
  - Deletion
- Prefer events over callbacks or hard dependencies
- Allow host apps to extend behavior cleanly

## Caching & Performance

- Cache content reads where appropriate
- Cache keys must be:
  - Namespaced
  - Configurable
- Never assume a specific cache driver

## Testing

- Use Orchestra Testbench
- Test:
  - Service provider boot
  - Route registration
  - API responses
  - Validation & authorization hooks
- Tests should target public API behavior
- Avoid application-specific assumptions

## Versioning & Stability

- Follow semantic versioning
- Treat:
  - Routes
  - Request payloads
  - Response shapes
    as public APIs
- Mark experimental endpoints clearly

## Output Rules

- Generate **package-level code only**
- Do not scaffold frontend, views, or auth
- Do not assume SPA or CMS UI
- Ask one clarification question if API intent is ambiguous

## What NOT To Do

- Do not generate Blade views or assets
- Do not assume authentication exists
- Do not hard-code User models
- Do not expose internal helpers or models
- Do not behave like an application

---
> Source: [pictastudio/contento](https://github.com/pictastudio/contento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
