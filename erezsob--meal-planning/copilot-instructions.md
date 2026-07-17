## meal-planning

> This is a meal planning app built with React, TypeScript, and Convex. The application provides a meal planning and shopping list.


# Cursor IDE Rules for Meal Planning App

## Project Context
This is a meal planning app built with React, TypeScript, and Convex. The application provides a meal planning and shopping list.

## Code Style & Standards
- Use pnpm to run npm commands and scripts (don't use yarn or npm).
- Follow the code quality standards in `.docs/CODE_STANDARDS.md`
- Use TypeScript with strict mode
- Prefer functional components and hooks
- Extract constants to `lib/constants.ts`
- Keep components under 300 lines when possible
- Refrain from using React's useEffect as much as possible
- Prefer handling effects on user interaction rather than relying on local state
- Prioritize semantic HTML and accessible UI
- When a function has more than 2 arguments, prefer passing an object
- Split big components into smaller components.
- `routes/index.tsx` should not have components, it should only have the route component.

## Architecture Patterns
- **Backend**: Convex for real-time document store + serverless functions
- **Frontend**: TanStack Start (file-based routing, React 19)
- **State**: Convex subscriptions for real-time sync, TanStack Query for caching
- **Styling**: Tailwind CSS v4 with mobile-first approach

## File Organization
- `convex/` - Backend schema, queries, mutations
- `lib/` - Shared utilities (fp.ts, errors.ts, constants.ts)
- `src/components/` - Reusable UI components
- `src/routes/` - TanStack file-based routes

## When Making Changes
- Update Convex schema → regenerate types with `npx convex dev`
- New routes → auto-generated in `src/routeTree.gen.ts`
- Test co-located with source (`.test.ts` / `.test.tsx`)
- Run `pnpm types:check` and `pnpm lint` before finishing your work.
- Run `pnpm test` to ensure all tests pass. (TBD: add test coverage)
- Run `pnpm test:coverage` to ensure all tests are covered. (TBD: add test coverage)

## Tech Stack
- TanStack Start (React 19, file-based routing)
- Convex (real-time backend)
- Tailwind CSS v4
- TypeScript strict mode
- Biome (formatting/linting)
- Vitest + React Testing Library

---
> Source: [erezsob/meal-planning](https://github.com/erezsob/meal-planning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
