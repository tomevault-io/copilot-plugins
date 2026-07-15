## family-tree

> This is a Next.js 14+ family tree application using:

# AI Collaboration Rules for Family Tree Application

## Project Context

This is a Next.js 14+ family tree application using:

- TypeScript
- Tailwind CSS
- Supabase for backend/auth
- Zustand for state management
- Shadcn/ui for components
- React Hook Form for forms

## Code Organization & Standards

### File Naming Conventions

- Pages: `kebab-case.tsx` (e.g., `family-members.tsx`)
- Components: `PascalCase.tsx` (e.g., `FamilyMemberCard.tsx`)
- Hooks: `use-kebab-case.ts` (e.g., `use-family-tree.ts`)
- Types: organized in `lib/types/index.ts`
- Constants: organized in `lib/constants/`
- Utilities: `kebab-case.ts` in `lib/utils/`

### Component Structure

- Keep components under 200 lines when possible
- Extract complex logic into custom hooks
- Use clear TypeScript interfaces for all props
- Follow pattern: imports → interfaces → component → export
- Use JSDoc comments for complex functions

### TypeScript Standards

- Define all interfaces in `lib/types/index.ts`
- Use strict typing, avoid `any`
- Create reusable type definitions
- Use enums from `lib/constants/enums.ts`
- Include proper typing for all Zustand stores

## Request Guidelines

### When Asking for Help Include:

1. **Context**: What you're trying to achieve
2. **Current state**: What exists now
3. **Desired outcome**: What you want to happen
4. **Constraints**: Any limitations or requirements
5. **File paths**: Specify exact files involved

### Error Reporting Format:

- Full error message and stack trace
- File/component where error occurs
- Code snippets (3-5 lines before/after issue)
- Expected vs actual behavior
- Browser console errors
- Network tab issues (for API problems)
- Current URL/route when issue occurs
- User authentication state (if relevant)

### Code Review Requests:

- Specify exact files to review
- Mention specific concerns (performance, security, maintainability)
- Include business logic context
- Ask for specific feedback areas

## Development Patterns

### State Management (Zustand)

- Keep store files focused on single concerns
- Use meaningful action names
- Include proper TypeScript typing
- Follow existing store patterns in `stores/` directory

### Database & API

- Follow RESTful conventions for API routes
- Use consistent error handling across all routes
- Include proper TypeScript types for database operations
- Use migrations for schema changes (in `migrations/` directory)
- Follow Supabase best practices

### Component Architecture

- Use existing component structure in `components/`
- Leverage shadcn/ui components from `components/ui/`
- Follow established patterns for modals, forms, and tables
- Use existing wrapper components (`auth-wrapper.tsx`, `frame-wrapper.tsx`)

### Styling

- Use Tailwind CSS classes
- Follow existing design patterns
- Leverage custom components in `components/ui/`
- Maintain responsive design principles

## Feature Development Workflow

1. Start with data flow analysis
2. Define TypeScript interfaces first
3. Create basic component structure
4. Add functionality incrementally
5. Handle edge cases and errors
6. Follow existing folder structure conventions

## Import Standards

```typescript
// Prefer explicit imports
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader } from "@/components/ui/card";

// Use path aliases (@/) for internal imports
// Group imports: external → internal → relative
```

## Performance Considerations

- Mention if components handle large datasets
- Specify real-time requirements
- Include context about user interactions
- Consider Next.js optimization patterns

## Authentication & Authorization

- Follow existing auth patterns in `app/(auth)/`
- Use Supabase auth helpers
- Respect user roles and permissions
- Handle auth state properly

## Testing & Debugging

- Test in multiple environments
- Include setup/data requirements
- Specify user roles/permissions needed
- Provide reproduction steps

## Simplicity Over Complexity

- Prefer simple, readable solutions over clever or complex ones
- Use built-in JavaScript/React features before reaching for external libraries
- Choose clear, explicit code over implicit or "magic" solutions
- Favor composition over inheritance
- Break down complex problems into smaller, manageable pieces
- Use straightforward naming that explains purpose clearly
- Avoid premature optimization - write simple code first, optimize later if needed
- When multiple approaches exist, choose the one that's easier to understand and maintain

## Code Quality

- Use consistent async/await patterns
- Include proper error boundaries
- Follow accessibility best practices
- Maintain existing code style
- Keep functions focused and pure when possible

## When Making Changes

- Always check existing similar implementations first
- Maintain consistency with current patterns
- Update related TypeScript types
- Consider impact on other components
- Test authentication flows if applicable
- Verify responsive design

## Collaboration Efficiency

- Use parallel tool calls when gathering information
- Provide complete context in single requests
- Specify exact file locations and line numbers when referencing code
- Include relevant business logic context
- Ask specific, actionable questions

---
> Source: [ThoseApp/family-tree](https://github.com/ThoseApp/family-tree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
