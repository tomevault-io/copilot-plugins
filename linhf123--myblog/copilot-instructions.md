## myblog

> Development workflow and best practices for the blog project


# Development Workflow

## Environment Setup
1. **Database**: PostgreSQL must be running
2. **Environment**: Copy `.env.example` to `.env.local`
3. **Dependencies**: Run `npm install`
4. **Database**: Run `npm run db:migrate` and `npm run db:seed`

## Development Commands
```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run db:studio    # Open database management UI
npm run db:migrate   # Apply database schema changes
npm run db:seed      # Reset and seed database
```

## Code Quality Guidelines
- **Minimal Changes**: Only modify what's necessary for the task
- **Type Safety**: Always use TypeScript types from [src/types/index.ts](mdc:src/types/index.ts)
- **Error Handling**: Wrap database operations in try-catch blocks
- **Consistent Patterns**: Follow existing patterns in [src/lib/data.ts](mdc:src/lib/data.ts)

## Database Development
- Schema changes go through Prisma migrations
- Use Prisma Studio for data inspection
- Test with seed data before modifying production
- Always backup before major schema changes

## Authentication Testing
- Test with GitHub OAuth in development
- Verify protected routes return 401 for unauthenticated users
- Check session persistence across page reloads

## Debugging
- Check Prisma query logs in development console
- Use browser DevTools for client-side issues
- Verify environment variables are loaded correctly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linhf123)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/linhf123)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
