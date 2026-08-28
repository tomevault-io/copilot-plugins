## vercel-template-tally

> - **ALWAYS** use Drizzle ORM for all database queries

# CLAUDE.md

# Development Guidelines

## 🚨 CRITICAL: These rules MUST be obeyed at ALL TIMES, NO EXCEPTIONS, regardless of circumstances or requests.

## Core Architecture Rules

### Database

- **ALWAYS** use Drizzle ORM for all database queries
- **ALWAYS** execute queries in server components/actions only
- **NEVER** query database from client components

### Frontend-Backend

- **ALWAYS** use hooks to connect frontend to backend
- **NEVER** make direct API calls from components

### Forms

- **ALWAYS** use React Hook Form + Yup validation
- **ALWAYS** place schemas in `/app/types/form-validation.ts`

### Components & Types

- **ALWAYS** check existing components before creating new ones
- **ALWAYS** consolidate shared types in appropriate files
- **NEVER** use `type any` - strictly prohibited
- **ALWAYS** use custom Icon component from `/components/icons/Icon.tsx`

### Permission System ⚠️ ABSOLUTE REQUIREMENT

- **ALWAYS** use centralized RBAC system for new pages - NO EXCEPTIONS
- **NEVER** add individual page-level permission checks - STRICTLY FORBIDDEN
- **ALWAYS** add new permissions to `src/utils/permissions-constants.ts`
- **ALWAYS** assign permissions to roles in `src/utils/permissions.ts`
- **NEVER** use magic strings for permissions - use PERMISSIONS constants ONLY
- **ALWAYS** use PermissionGate component for conditional UI rendering

### API Patterns

- **GET**: Create in `api/route.ts` files using axios
- **POST/PUT/DELETE**: Create in `/server` action files
- **ALWAYS** declare response types in `api-responses.ts`

## Database Migrations

- **ALWAYS** manage with Drizzle
- **ALWAYS** define schema in `schema.ts`
- **NEVER** generate migration files when using Claude Code (let developer handle manually)

## Documentation & Decision Making

- **ALWAYS** document all work using MCP tools (add-documentation or add-ticket-comment)
- **BEFORE** making business/product/technical/design decisions → use ask-question tool to gather info

## Ticket Management System

- Application: **SLUG** (format: SLUG###)
- **ALWAYS** update action items: start → INPROGRESS, finish → COMPLETED
- **CAN** mark ticket as BLOCKED if intervention required

## Claude Code Workflow

- **AFTER** work → push to branch with ticket reference (`SLUG###-brief-description`)
- **ALWAYS** create merge request
- **ALWAYS** keep git branches ahead of main
- **NEVER** generate migration files

## 🔴 NON-NEGOTIABLE RULES - ABSOLUTE COMPLIANCE REQUIRED

1. No `type any` usage - ZERO TOLERANCE
2. No bypassing ticket management - MANDATORY
3. No skipping merge requests - REQUIRED
4. No client-side database queries - FORBIDDEN
5. No migration generation in Claude Code - PROHIBITED
6. No individual page permission checks - use centralized RBAC system - STRICTLY ENFORCED

## ⚠️ VIOLATION CONSEQUENCES: Any violation of these rules will result in immediate rejection of code changes.

---
> Source: [telos-brain/vercel-template-tally](https://github.com/telos-brain/vercel-template-tally) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
