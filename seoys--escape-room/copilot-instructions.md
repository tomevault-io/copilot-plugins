## escape-room

> - Use React Server Components by default


# UI Components & Styling Rules

## Component Architecture

- Use React Server Components by default
- Add `'use client'` only when client-side state or interactivity is needed
- Prefer composition over inheritance
- Keep components focused and single-purpose

## Styling Guidelines

- Use Tailwind CSS utility classes for styling
- Custom styles go in [src/app/globals.css](mdc:src/app/globals.css)
- Use the Gaegu font family as defined in [src/app/layout.tsx](mdc:src/app/layout.tsx)
- Maintain responsive design for mobile and desktop

## Component Structure

- [src/components/Card.tsx](mdc:src/components/Card.tsx) - Reusable card component
- Room-specific components should be colocated with their pages
- Shared UI components go in [src/components/](mdc:src/components/)

## Accessibility & UX

- Use semantic HTML elements
- Include proper ARIA labels for interactive elements
- Ensure keyboard navigation works properly
- Provide clear visual feedback for user actions
- Use Korean language (lang="ko") as specified in layout

## Image & Media Handling

- Prefer WebP format for images in [public/images/](mdc:public/images/)
- Use Next.js Image component for optimized loading
- Background images defined in [tailwind.config.js](mdc:tailwind.config.js)
- Game assets stored in [public/assets/](mdc:public/assets/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seoys)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/seoys)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
