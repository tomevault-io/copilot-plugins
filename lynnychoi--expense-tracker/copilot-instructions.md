## expense-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Korean household expense tracking application - a single-page app for collaborative financial management between household members (max 7 per household). Built as a modern web application with real-time collaboration and Korean Won (₩) currency support.

## Commands

### Development
```bash
npm run dev          # Start development server (localhost:3000)
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint - always run before commits
npm run typecheck    # Run TypeScript compiler check - ensure type safety
```

**Error Handling Protocol**: When any error occurs during development:
1. **Inspect and Document**: Analyze the error thoroughly and document what went wrong
2. **Present Options**: Provide multiple resolution approaches with trade-offs
3. **Recommend Best Option**: Suggest the optimal solution based on project context
4. **Wait for Approval**: Always wait for user confirmation before implementing fixes

### Visual Testing (Required for Frontend Changes)
After any frontend implementation, immediately run visual verification:
```bash
# Run Playwright in headed mode for visual inspection
npx playwright test --headed
```

### Git Workflow
Use [gitmoji.dev](http://gitmoji.dev) conventions for commit messages. Group related changes logically:
```bash
git add .
git commit -m "✨ feat: add Korean Won currency utilities"
git commit -m "♻️ refactor: organize transaction types in shared file"
git commit -m "🔧 chore: configure Shadcn/UI components"
```

### Shadcn/UI Components
```bash
npx shadcn@latest add [component]  # Add new Shadcn/UI components
```
Use `mcp__shadcn__getComponents` to discover available components and `mcp__shadcn__getComponent` for detailed implementation guidance before adding manually.

### Database (Requires Docker Desktop)
```bash
supabase start       # Start local Supabase (requires Docker)
supabase db reset    # Reset local database and apply migrations
supabase db push     # Push schema changes to remote
supabase status      # View connection details
```

## Architecture

### Tech Stack
- **Frontend**: Next.js 15 App Router, TypeScript, Tailwind CSS v4
- **UI**: Shadcn/UI components (configured for New York style)
- **Backend**: Supabase (PostgreSQL + Auth + Storage)  
- **Currency**: Korean Won (₩) stored as integers
- **AI**: OpenAI API integration planned

### Key Architectural Decisions

**Single Page Application**: All functionality lives on one main dashboard page (`src/app/page.tsx`) with modal dialogs for forms. This design prioritizes simplicity and real-time collaboration. After any change to the main dashboard, immediately test visual compliance across desktop, tablet, and mobile viewports using Playwright in headed mode.

**Korean Won Currency System**: All amounts stored as integers without decimals. Always use the established utilities in `src/lib/currency.ts` for consistency:
- `formatKRW(1234567)` → `"₩1,234,567"`
- `parseKRW("₩1,234,567")` → `1234567`
- `isValidKRWAmount()` for validation before storage

**Multi-household Architecture**: Users belong to one household at a time (max 7 members). Household creators have admin privileges. Data access controlled via Supabase Row Level Security.

**Component Structure**: Shadcn/UI components in `src/components/ui/` with path aliases configured (`@/components`, `@/lib`, etc.). Form validation with React Hook Form + Zod. Follow existing patterns when adding new components - maintain consistent import styles, use TypeScript strictly, and follow the established folder structure.

**Frontend Change Protocol**: IMMEDIATELY after implementing any UI change, perform visual verification:
1. Identify what changed - Review modified components/pages
2. Navigate to affected pages - Use `mcp__playwright__browser_navigate` to visit each changed view  
3. Capture screenshots - Take full page screenshots at desktop, tablet, mobile viewports
4. Verify design compliance - Compare against existing design patterns and UX best practices
5. Validate feature implementation - Ensure the change fulfills the user's specific request
6. Check acceptance criteria - Review any provided context files or requirements against PRD.md user stories
7. Check for errors - Run `mcp__playwright__browser_console_messages` and resolve if necessary
8. Update PRD status - Mark completed user stories in PRD.md, update specifications if requirements changed

**Error Resolution Process**: If errors are found during visual testing:
- Document the specific error with screenshots and console messages
- Identify root cause (CSS, JavaScript, API, database, etc.)
- Present 2-3 resolution options with pros/cons for each approach
- Recommend the best solution and wait for user approval before proceeding

**MCP Integration**: Leverage connected MCPs for development tasks - use Shadcn MCP for component discovery (`mcp__shadcn__getComponents`), GitHub MCP for repository management, and Vercel MCP for deployment operations rather than manual processes.

## Core Domain Models

### Transaction System
**Transaction Types**: `expense` | `income` with Korean Won amounts (integers only)
**Person Attribution**: Each transaction assigned to household member OR "household" for shared items
**Tags & Colors**: Each tag gets assigned a color from 30-color palette for visual distinction
**Recurring Transactions**: Monthly/yearly auto-generation with 7-day advance notifications

### Household Management  
**Multi-user**: 1-7 members per household, creator has admin privileges (invite/remove members)
**Data Isolation**: Row Level Security ensures members only see their household's data
**Soft Deletes**: Households appear deleted but data retained in backend

### Budget & Analytics
**Per-tag Goals**: Monthly budget limits set for each category/tag
**Real-time Tracking**: Progress indicators (red/yellow/green) based on spending vs goals
**Comparison Views**: Month-over-month, year-over-year analysis
**Export Options**: CSV data export, PDF reports with charts

## Development Context

### Current State
Foundation phase complete with Next.js 15, Shadcn/UI (New York style), TypeScript strict mode, and Korean Won utilities. See `PLAN.md` for 22-week development roadmap across 10 phases. When implementing new features, follow the established patterns in existing code and maintain the high quality standards set by the foundation.

**Commit Strategy**: Group related changes logically between todos - for example, commit all authentication setup files together (🔐 auth: setup Supabase auth flow), then all form components together (✨ feat: add transaction form components), rather than mixing concerns in single commits. Always include PRD.md updates when completing user stories or changing specifications.

### Type System
Comprehensive TypeScript definitions in `src/types/index.ts` cover all domain models. Korean Won amounts always typed as `number` (integers). Form data uses string inputs that get parsed to numbers. Always use existing types and extend them when needed rather than creating new interfaces. Maintain strict TypeScript compliance - the `noUncheckedIndexedAccess` flag is enabled.

### Project Documents
- `PRD.md`: Complete product requirements with 30 indexed user stories (US-001 to US-030) - **MUST be updated for any spec changes**
- `PLAN.md`: Detailed 22-week development plan across 10 phases
- Current implementation follows Phase 1 (Foundation & Infrastructure)

**PRD Maintenance**: Any change in specifications requires immediate PRD updates (add/modify/delete user stories). Track implementation status by updating completion markers in PRD.md rather than maintaining separate tracking systems.

**Todo Implementation**: Each todo should result in 1-3 logical commits using gitmoji conventions. Leverage MCP tools throughout - use GitHub MCP for repository operations, Vercel MCP for deployment tasks. **Update PRD.md completion status** after implementing each user story. Example for "Create core database schema":
1. `🗃️ db: create user and household tables`
2. `🗃️ db: add transaction and tag tables with relationships`  
3. `🔒 security: implement Row Level Security policies`
4. `📝 docs: mark relevant user stories complete in PRD.md`

## Environment Setup

Required environment variables:
```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=  
SUPABASE_SERVICE_ROLE_KEY=
OPENAI_API_KEY=
```

**MCP-Enhanced Workflow**: 
- Use `mcp__vercel__deploy_to_vercel` for deployment instead of manual Vercel CLI
- Use `mcp__github__create_pull_request` and related GitHub MCP tools for repository management
- Use `mcp__vercel__search_vercel_documentation` for deployment guidance
- Check `mcp__shadcn__getComponents` before manually searching for UI components
- Use `mcp__playwright__browser_navigate` and `mcp__playwright__browser_console_messages` for immediate visual verification of frontend changes

---
> Source: [lynnychoi/expense-tracker](https://github.com/lynnychoi/expense-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
