## react-development

> Developing React components in Astro application

## Frontend

### Guidelines for React

#### React Coding Standards

- Use functional components with hooks instead of class components
- Implement React.memo() for expensive components that render often with the same props
- Utilize React.lazy() and Suspense for code-splitting and performance optimization
- Use the useCallback hook for event handlers passed to child components to prevent unnecessary re-renders
- Prefer useMemo for expensive calculations to avoid recomputation on every render
- Prefer relying on Tailwind media queries instead of manual style recalculations
- If there's a need to attach React components to Astro pages and make them browser-first (i.e. using window), use client:only directive to make the component exclusively run on the client.
- Use Tailwind responsive variants (sm:, md:, lg:, etc.) for adaptive designs - under no circumstances calculate this manually

---
> Source: [przeprogramowani/ai-rules-builder](https://github.com/przeprogramowani/ai-rules-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
