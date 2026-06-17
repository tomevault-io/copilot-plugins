## 20-frontend-nextjs

> Next.js frontend rules


Frontend stack:
- Next.js
- TypeScript
- Component-driven UI
- API integration with backend services

Frontend rules:
- Prefer TypeScript for all frontend logic.
- Keep components small, reusable, and focused.
- Keep presentation separate from business/data-fetching logic.
- Use hooks, services, or server actions appropriately based on project style.
- Handle loading, error, and empty states explicitly.
- Preserve design consistency and naming consistency.

Preferred frontend structure:
- frontend/app/
- frontend/components/
- frontend/features/
- frontend/hooks/
- frontend/lib/
- frontend/services/
- frontend/types/
- frontend/tests/

UI/UX rules:
- Use accessible markup and semantic HTML.
- Prefer reusable design system components.
- Avoid oversized page components.
- Avoid embedding complex API logic directly inside visual components unless already established in the repo.

State/data rules:
- Keep API clients centralized.
- Normalize repeated API access patterns.
- Handle auth state carefully.
- Validate critical inputs on both client and server.

Do not:
- Mix many responsibilities into one component.
- Hardcode API URLs in components.
- Duplicate UI patterns that should be shared.

---
> Source: [aiagentwithdhruv/laptop-finder-ai](https://github.com/aiagentwithdhruv/laptop-finder-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
