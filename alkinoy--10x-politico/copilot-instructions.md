## 10x-politico

> SpeechKarma is a web application that captures and publishes public statements made by politicians. Authenticated users can submit statements (who/when/what). Anyone can browse recent statements and review each politician’s timeline to understand positions over time.

# AI Rules for SpeechKarma

SpeechKarma is a web application that captures and publishes public statements made by politicians. Authenticated users can submit statements (who/when/what). Anyone can browse recent statements and review each politician’s timeline to understand positions over time.

## Tech Stack

- Astro 5
- TypeScript 5
- React 19
- Tailwind 4
- Shadcn/ui

## Project Structure

When introducing changes to the project, always follow the directory structure below:

- `./.md` - All documentation files (*.md) belong here
- `./src` - source code
- `./src/layouts` - Astro layouts
- `./src/pages` - Astro pages
- `./src/pages/api` - API endpoints
- `./src/middleware/index.ts` - Astro middleware
- `./src/db` - Supabase clients and types
- `./src/types.ts` - Shared types for backend and frontend (Entities, DTOs)
- `./src/components` - Client-side components written in Astro (static) and React (dynamic)
- `./src/components/ui` - Client-side components from Shadcn/ui
- `./src/lib` - Services and helpers 
- `./src/assets` - static internal assets
- `./public` - public assets

When modifying the directory structure, always update this section.

**Note:** All markdown documentation files (implementation summaries, API documentation, testing guides, etc.) should be stored in the `./.md` directory to keep the root directory clean and organized.

## Coding practices

### Guidelines for clean code

- Use feedback from linters to improve the code when making changes.
- Prioritize error handling and edge cases.
- Handle errors and edge cases at the beginning of functions.
- Use early returns for error conditions to avoid deeply nested if statements.
- Place the happy path last in the function for improved readability.
- Avoid unnecessary else statements; use if-return pattern instead.
- Use guard clauses to handle preconditions and invalid states early.
- Implement proper error logging and user-friendly error messages.
- Consider using custom error types or error factories for consistent error handling.
- Always run lint to fix errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alkinoy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
