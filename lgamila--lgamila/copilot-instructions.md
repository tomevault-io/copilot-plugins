## 020-codebase-structure

> The project follows a monorepo structure using pnpm workspaces:

# Codebase Structure and Commands

## Project Structure

The project follows a monorepo structure using pnpm workspaces:

### Core Directories
- `apps/` - Main applications
  - `backend/` - Backend service with the following structure:
    - `src/` - Source code
      - `api/` - API routes and handlers
      - `lib/` - Core libraries and utilities
        - `auth/` - Authentication related code
        - `cache/` - Caching mechanisms
        - `db/` - Database interactions
        - `errors/` - Error handling
        - `middleware/` - Request middleware
        - `repositories/` - Data access layer
        - `services/` - Business logic
      - `migrations/` - Database migrations
      - `scripts/` - Utility scripts
      - `types/` - TypeScript type definitions
    - `drizzle/` - Database schema and migrations
    - `static/` - Static assets
  - `client/` - Web client application
    - `src/` - Source code
      - `assets/` - Static assets
      - `components/` - React components
        - `ui/` - Reusable UI components
      - `hooks/` - Custom React hooks
      - `lib/` - Utilities and helpers
        - `guards/` - Route guards and protection
        - `mutations/` - API mutation hooks
        - `queries/` - API query hooks
        - `utils/` - General utilities
      - `pages/` - Page components
      - `types/` - TypeScript type definitions
    - `public/` - Public assets
- `packages/` - Shared packages and utilities
  - `shared/` - Shared utilities and type definitions
    - `src/`
      - `types/` - Shared type definitions
      - `validation/` - Shared validation schemas
  - `eslint-config/` - Shared eslint configuration
  - `typescript-config/` - Shared typescript configuration
- `.cursor/` - Project rules and documentation
- `.vscode/` - VS Code configuration
- `.turbo/` - Turborepo cache and configuration

### Configuration Files
- `package.json` - Project dependencies and scripts
- `pnpm-workspace.yaml` - Workspace configuration
- `turbo.json` - Turborepo configuration
- `.npmrc` - NPM configuration
- `.gitignore` - Git ignore configuration

## Available Commands

### Development Commands
```bash
# Start development servers
pnpm dev                    # Start all development servers
pnpm --filter backend dev   # Start backend development server
pnpm --filter client dev    # Start client development server

# Code Quality
pnpm lint                  # Run linting
pnpm format               # Format code
pnpm check-types         # Type check all packages
```

### Database Commands (Backend)
```bash
pnpm --filter backend db:push      # Push schema to database
pnpm --filter backend db:generate  # Generate migrations from schema changes
pnpm --filter backend db:migrate   # Apply migrations to database
pnpm --filter backend db:studio    # Launch Drizzle Studio
```

### Build Commands
```bash
pnpm build                 # Build all packages
pnpm --filter client build       # Build only client
```

### Package Management
```bash
# Add dependencies to specific packages
pnpm add <package> --filter <package-name>      # Add production dependency
pnpm add -D <package> --filter <package-name>   # Add dev dependency
```

## Development Setup

1. Install dependencies:
   ```bash
   npm i -g pnpm
   pnpm install
   ```

2. Configure environment:
   - Create appropriate environment files for each app
   - Fill required environment variables

3. Start development:
   ```bash
   # Start all apps
   pnpm dev
   
   # Or start individual apps
   pnpm --filter backend dev
   pnpm --filter client dev
   ```

4. Build for production:
   ```bash
   pnpm build
   ```

## Best Practices

1. Always use pnpm for package management
2. Use the `--filter` flag when working with specific packages
3. Keep shared code in the `packages/shared` directory
4. Follow the established environment variable conventions
5. Run code quality checks before committing:
   ```bash
   pnpm lint
   pnpm check-types
   ``` 

---
> Source: [LGamila/lgamila](https://github.com/LGamila/lgamila) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
