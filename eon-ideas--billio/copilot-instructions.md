## billio

> Billio is an invoice management application with role-based access control, built with Vue 3, TypeScript, Tailwind CSS, and Supabase.


# Billio - Windsurf Global Rules

## Project Overview

Billio is an invoice management application with role-based access control, built with Vue 3, TypeScript, Tailwind CSS, and Supabase.

## Technical Stack

### Frontend
- Framework: Vue 3
- Language: TypeScript
- Styling: Tailwind CSS
- State Management: Pinia
- Router: Vue Router

### Backend
- Database: Supabase
- Authentication: Supabase Auth
- Hosting: Netlify

## Code Conventions

### General
- Language: English
- Indentation: 2 spaces
- Maximum line length: 100 characters
- Use single quotes for strings
- Always use semicolons

### TypeScript
- Strict mode enabled
- Avoid using `any` type
- Prefer interfaces over types where appropriate
- Use proper type annotations for all variables, parameters, and return types

### Vue
- Use Composition API with script setup syntax
- Name components using PascalCase
- Always include prop validation
- Keep component files focused on a single responsibility
- Extract reusable logic into composables

### File Structure
- Views: `src/views`
- Components: `src/components`
- Stores: `src/stores`
- Composables: `src/composables`
- Types: `src/types`

## Key Features

### Role-Based Access Control
- Two roles: ADMIN and USER
- ADMIN has full CRUD access to all database tables
- USER has limited access based on role permissions
- UI must include purple ADMIN badge for admin users

### Currency Exchange
- Exchange rate calculation for non-EUR currencies
- Automatic rate fetching from Billio backend API
- Display conditional input field in InvoiceForm when needed
- Exchange rate defined as value of 1 unit customer currency in EUR

## Database Structure and Permissions

### Tables and Access
- company_info: ADMIN (CRUD), USER (READ)
- email_templates: ADMIN (CRUD), USER (READ)
- customers: ADMIN (CRUD), USER (CRUD)
- invoices: ADMIN (CRUD), USER (CRUD)
- invoice_items: ADMIN (CRUD), USER (CRUD)
- user_roles: ADMIN (CRUD), USER (NONE)

### Security
- Row Level Security (RLS) policies must be implemented for all tables
- Enforce role-based permissions consistently

## Development Workflow

### Commands
- Development: `npm run dev`
- Build: `npm run build`
- Preview: `npm run preview`
- Type checking: `npm run type-check`

### Feature Implementation Guidelines
- Pre-Implementation:
  - Review README.md and PRD.md for existing feature documentation and context
  - Understand how new features integrate with existing functionality
  - Check for related features in the codebase
- Post-Implementation:
  - Update PRD.md with new feature documentation
  - Include implementation details in technical documentation
  - Update any affected existing documentation

## Documentation Requirements

Always adhere to these documentation rules:
- Update PRD.md after implementing new features
- Consult PRD.md before implementing new features
- Keep documentation up-to-date with the codebase
- Document changes to database schema
- Document changes to user roles and permissions

## Files to Ignore
- node_modules
- .temp
- dist
- .env
- *.log

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eon-ideas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
