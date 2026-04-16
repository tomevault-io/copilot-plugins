## men-in-the-arena

> description: React + Next.js Standards

---
description: React + Next.js Standards
globs: "*.js,*.jsx,*.ts,*.tsx"
---
# React + Next.js Standards

<rule>
name: react_standards
description: Core coding conventions for React & Next.js
filters:
  - type: file_extension
    pattern: "\\.(js|jsx|ts|tsx)$"
  - type: event
    pattern: "file_create|file_modify"

actions:
  - type: suggest
    message: |
      ***React + Next.js Best Practices***

      **Component Structure**
      • Use functional components with hooks (no legacy class components).
      • Keep components small, focusing on one responsibility.
      • Separate UI from complex logic; place heavier logic in custom hooks or utility functions.

      **Styling & Layout**
      • For styling, prefer Tailwind classes or CSS modules (avoid inline styles).
      • Keep layout components in `layouts/` or `components/` if reused across pages.

      **Props & State Management**
      • Use TypeScript interfaces/types for all props and state.
      • Avoid duplicating global state if local component state suffices.
      • If you need cross-component state, consider React Context or a library like Zustand.

      **Data Fetching**
      • For SSR/SSG, use Next.js data fetching methods (`getServerSideProps`, `getStaticProps`).
      • For client-side fetching, rely on React Query or fetch within `useEffect` if minimal.
      • Keep fetch logic DRY in separate utilities or hooks (e.g., `useUserProfile()`).

      **Routing & Pages**
      • Put top-level routes in `app/` (Next.js 13+ file-based routing) or `pages/` (Classic).
      • Keep your file/folder names descriptive (e.g. `pages/profile/[id].tsx`).

      **Performance**
      • Use React’s memoization hooks (`useMemo`, `useCallback`) for expensive computations.
      • Code-split large dependencies via dynamic imports when appropriate.
      • Mind SSR performance—avoid blocking calls in server components.

metadata:
  priority: high
  version: 1.0
</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yashchitneni) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
