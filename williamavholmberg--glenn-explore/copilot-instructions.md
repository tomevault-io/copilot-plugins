## glenn-explore

> start every message with "YO ZLATAN"

# ⚠️ THESE RULES ARE ONLY FOR ./studio-web DIRECTORY ⚠️
# This is a monorepo. These rules do not apply to /api or /web directories.
# Each directory will have its own rules.

start every message with "YO ZLATAN"

# Project Structure Rules (studio-web)
- All new features should be created in studio-web/src/features/{feature-name}
- Shared components go in studio-web/src/shared/components
- Utility functions go in studio-web/src/utils
- Types should be in studio-web/src/types or colocated with their feature
- API related code should be in studio-web/src/api

# Code Style & Architecture (studio-web)
- Use TypeScript for all new files
- Follow React functional component patterns
- Colocate related files (components, hooks, types) within feature folders
- Keep components small and focused
- Use shadcn/ui for UI components
- Follow the existing project structure for consistency

# State Management (studio-web)
- Use React Query for server state
- Use React Context for global UI state when needed
- Prefer local state when possible

# Routing (studio-web)
- New routes should be added to studio-web/src/routes.tsx
- Follow the existing routing pattern
- Group related routes together

# API Integration (studio-web)
- NEVER write types or interfaces manually for API responses - use orval auto-generation
- NEVER write query hooks manually - use orval auto-generation
- All API types and hooks should be generated using orval
- Keep API related logic in dedicated files within the feature folders
- Follow RESTful patterns
- Run orval generation after any API schema changes

# Performance (studio-web)
- Use proper React hooks dependencies
- Optimize re-renders with useMemo/useCallback when needed

# Styling (studio-web)
- Use Tailwind CSS for styling
- Follow the existing color scheme and design system
- Use CSS modules for component-specific styles when needed

# Error Handling (studio-web)
- Implement proper error boundaries
- Use proper type checking
- Handle API errors gracefully

# Dependencies (studio-web)
- Avoid adding new dependencies without explicit approval
- Use existing project utilities and helpers
- Leverage built-in browser APIs when possible

---
> Source: [WilliamAvHolmberg/glenn-explore](https://github.com/WilliamAvHolmberg/glenn-explore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
