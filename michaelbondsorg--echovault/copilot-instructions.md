## echovault

> Always use PascalCase (e.g., UserGreeting.jsx, ProfileBadge.jsx).

# Adobe Enablement - EchoVault Coding Standards

## 1. Naming Conventions

### React Components
Always use PascalCase (e.g., UserGreeting.jsx, ProfileBadge.jsx).

### Hooks
Always use camelCase starting with "use" (e.g., useAuthStore.js).

### Folders
Use kebab-case (e.g., components/user-profile/).

### Files
Ensure the file name matches the primary export (e.g., src/components/zen/TopBar.jsx contains export const TopBar).

## 2. Design System (Glassmorphism)

When creating or modifying UI elements in src/components/zen, refer to the styling patterns in GlassCard.jsx.

Use Tailwind CSS classes that promote transparency and blur: bg-white/10, backdrop-blur-md, and border-white/20.

## 3. Import Organization

Follow this order for all imports to keep the code clean:

1. React core imports (useState, useEffect, etc.)
2. Third-party libraries (e.g., lucide-react, zustand)
3. Internal stores (e.g., @/stores/authStore)
4. Internal components
5. Local styles/utilities

## 4. State Management (Zustand)

When accessing user data, prioritize using the authStore found in src/stores/authStore.js.

Example: `const user = useAuthStore((state) => state.user);`

## 5. TypeScript & Best Practices

- Prefer functional components with arrow functions.
- Always provide descriptive comments for new logic blocks using JSDoc style.
- Use try/catch blocks for any logic involving store hydration or external API calls.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelBondsorg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
