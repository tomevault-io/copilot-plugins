## ninetyninety

> Always include [PROJECT.md](mdc:PROJECT.md) and [README.md](mdc:README.md) in your context of rules.

Always include [PROJECT.md](mdc:PROJECT.md) and [README.md](mdc:README.md) in your context of rules.

## Important Project Constraints

### UI/CSS Framework
- **DO NOT USE TAILWIND CSS** under any circumstances
- We are exclusively using **Chakra UI v3** for all styling and components
- No utility-first CSS frameworks should be suggested or implemented
- All styling should be done through Chakra UI components and its theming system

### Technology Stack Requirements
- **Package Manager**: Bun (not npm or yarn)
- **Frontend Framework**: Next.js 14+ with App Router
- **UI Library**: Chakra UI v3 only
- **Database**: Supabase (PostgreSQL)
- **Language**: TypeScript
- **Code Quality**: Biome for linting and formatting

### Development Principles
- Follow Chakra UI's component-based approach for all UI elements
- Use Chakra UI's responsive design system (breakpoints, responsive arrays)
- Leverage Chakra UI's theming system for consistent design
- Prioritize accessibility through Chakra UI's built-in ARIA support

### Testing Strategy
- Unit Testing: Bun's built-in test runner
- E2E Testing: Playwright
- No Jest or other testing frameworks

## Code Style Guidelines
- Use TypeScript for all files
- Follow Biome's default configuration for linting and formatting
- Use Chakra UI's design tokens for spacing, colors, and typography
- Implement responsive design using Chakra UI's responsive syntax

## Project Structure
- Use Next.js App Router structure
- Components should be Chakra UI-based React components
- Database interactions through Supabase client
- API routes for server-side operations


## Useful commands:

Can use the gh cli to create a new branch and push it to the remote repository.

```bash
gh pr create --title "Add new feature" --body "This is a new feature"
```

Can use the gh cli to watch the status of the PR.

```bash
gh run watch
```

Can use the gh cli to merge a PR.
```bash
gh pr merge --auto
```


*These guidelines ensure consistency and prevent the introduction of conflicting technologies.*

# NinetyNinety Project Rules

## Technology Stack
- **Package Manager**: Bun (NOT npm or yarn)
- **Frontend Framework**: Next.js 14+ with App Router
- **UI Library**: Chakra UI v3 ONLY (NO Tailwind CSS)
- **Database**: Supabase (PostgreSQL) with Prisma ORM
- **Language**: TypeScript
- **Code Quality**: Biome for linting and formatting
- **Testing**: Bun (unit) + Playwright (E2E)

## CRITICAL RULES
1. **NEVER use Tailwind CSS** - We exclusively use Chakra UI v3
2. **ALWAYS use Bun** commands (not npm or yarn)
3. **ALWAYS use Chakra UI components** for all UI elements
4. **FOLLOW the existing code patterns** in the project

## Code Style
- Use TypeScript for all files
- Follow Biome's configuration for formatting
- Use Chakra UI's design tokens for spacing, colors, and typography
- Implement responsive design using Chakra UI's responsive syntax

## Common Commands
# Development
bun dev                  # Start dev server on port 3009
bun build               # Build for production
bun start               # Start production server

# Code Quality
bun check               # Run Biome checks
bun check:fix           # Fix linting/formatting issues
bun format              # Format code

# Testing
bun test                # Run unit tests
bun test:e2e            # Run E2E tests

# Database (Prisma)
bun db:push             # Sync schema with database
bun db:migrate          # Run migrations
bun db:studio           # Open Prisma Studio

# General guidelines

## When You Make Mistakes

- I'll let you know.
- when we figure out the correct thing to do offer to add a note about it to this document so we can prevent that in the future.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/foxyblocks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
