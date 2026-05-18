## project-patterns

> - React components are placed in `src/components/`

# Project Patterns for Personal Finance Dashboard

## Code Organization
- React components are placed in `src/components/`
- Page components are in `src/app/` following Next.js App Router conventions
- API routes are defined in `src/app/api/` directory
- Utility functions are placed in `src/lib/`
- Types are defined in `src/types/`

## Naming Conventions
- React components use PascalCase
- Files containing React components also use PascalCase
- Utility functions and files use camelCase
- API routes use kebab-case for directories and files

## State Management
- React Query is used for server state management
- React hooks are used for component-level state
- Environment variables for configuration
- Database as source of truth for financial data

## Component Structure
- UI components primarily use shadcn/ui library
- Components should be focused on specific functionality
- Common layouts and patterns should be reused
- TailwindCSS for styling with className approach

## API Integration 
- API credentials stored in environment variables
- Financial service tokens stored in database
- Plaid Link used for bank connection UI
- OAuth flow used for Coinbase integration
- Account data normalized across providers

## Data Handling
- Historical balance data tracked over time
- SQLite database for data persistence
- Prisma for database access
- Timestamps for all financial data points
- Regular refresh via cron job

## Error Handling
- API errors should be properly caught and logged
- User-friendly error messages for connection issues
- Retry mechanisms for transient failures
- Detailed logs for debugging

## Security Practices
- No sensitive credentials in client-side code
- Option to mask sensitive financial information in UI
- Environment variables for sensitive configuration
- No exposure of raw API responses to frontend

## CI/CD & Deployment
- Self-hosted application
- Manual deployment process
- Cron job for automated data refresh
- Logging to track automated processes

## User Experience
- Clean, minimal UI
- Focus on data visualization
- Account customization (hiding, nicknames)
- Responsive design for mobile and desktop

---
> Source: [dotnetfactory/personal-financial-dashboard](https://github.com/dotnetfactory/personal-financial-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
