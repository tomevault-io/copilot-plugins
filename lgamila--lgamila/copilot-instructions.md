## 030-coding-style

> Coding Style: Guidelines for code formatting, naming conventions, and best practices


# Coding Style Guidelines

## Technology-Specific Guidelines

### TypeScript
- Use TypeScript strictly - avoid `any` types
- Use interfaces for object shapes that will be extended
- Use type aliases for complex types and unions
- Use the latest TypeScript features appropriately

### React Components
- Use functional components with hooks
- Use named exports for components
- Keep components small and focused on a single responsibility
- Use proper prop typing

### Backend (Hono)
- Use middleware for cross-cutting concerns
- Validate request data with Zod
- Use proper error handling with structured responses
- Structure routes logically by resource

### Browser Extension (Plasmo)
- Follow Plasmo framework conventions
- Separate content scripts from background logic
- Keep performance in mind for extension code

## Comments and Documentation
- Write self-documenting code, no need to add extra comments
- Add comments for complex logic
- Document public APIs and interfaces
```typescript
/**
 * Fetches user data and handles pagination
 * @param page - The page number to fetch
 * @param limit - Number of items per page
 * @returns Paginated user data
 */
async function fetchUsers(page: number, limit: number): Promise<PaginatedResponse<User>> {
  // ...
}
```

## Styling
- Use Tailwind CSS for styling
- Follow component-based styling practices
- Use utility classes for one-off styling needs
- Extract common patterns to shared components

## Git Commit Guidelines

### Commit Messages
- Use conventional commits format
- Include ticket number when applicable
- Keep messages clear and concise
```bash
# Format: <type>(<scope>): <description>
feat(auth): add social login support
fix(api): handle pagination edge cases
chore(deps): update dependencies
```

### Branch Names
- Use feature/, bugfix/, hotfix/ prefixes
- Include ticket number when applicable
- Use kebab-case
```bash
feature/KR-123-user-authentication
bugfix/api-pagination
hotfix/login-crash
```

## Core Philosophy

- Code is primarily written for humans to read and understand, not just for computers to execute
- "Explicit is better than implicit. Simple is better than complex. Readability counts."
- Maintain consistency with existing code style when modifying files
- Keep code DRY (Don't Repeat Yourself)
- Prefer verbose variable names and maintainability over concise code

---
> Source: [LGamila/lgamila](https://github.com/LGamila/lgamila) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
