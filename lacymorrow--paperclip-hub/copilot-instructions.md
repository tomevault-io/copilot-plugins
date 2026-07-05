## react

> React.js Best Practices


# React Best Practices

Use functional components and hooks for state management.
Ensure components are reusable and maintainable.
Prefer React Server Components for fetching data.
Prefer server actions over API requests for mutating data.
Maintain a separation of concerns between client and server components.

Prefer arrow functions for React components:
✅ export const Component = () => { ... }
❌ export function Component() { ... }
❌ export default function Component() { ... }

---
> Source: [lacymorrow/paperclip-hub](https://github.com/lacymorrow/paperclip-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
