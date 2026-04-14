## portfolio-website

> - Vite + React 19 + TypeScript

# Agent Guidelines

## Stack
- Vite + React 19 + TypeScript
- Tailwind CSS v4
- Framer Motion (for animations)
- Radix UI (for accessible primitives)
- React Router v7

## Conventions
- Functional components only, no class components.
- Always use TypeScript interfaces, not types, for props.
- Components live in src/components/
- Pages live in src/pages/
- Use the `cn()` utility for merging Tailwind classes.
- Prioritize Lucide React for icons.
- Use 'motion' from the `motion` package for animations.

## Do Not Touch
- vercel.json
- vite.config.ts
- .github/workflows/
- package.json (unless adding a specific dependency)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cesargarciari) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
