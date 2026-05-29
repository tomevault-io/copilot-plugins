## nasserxx-github-io

> - Keep components small and focused

# Cursor Rules for CV Website

# Code Organization
- Keep components small and focused
- Use TypeScript interfaces for all props and data structures
- Follow the DRY principle (Don't Repeat Yourself)
- Keep files under 200 lines where possible
- Name components clearly according to their purpose

# Styling Rules
- Use Tailwind CSS classes for styling where possible
- Use CSS variables for theme colors in globals.css
- Use responsive design patterns with mobile-first approach
- Use dark mode variants for all colors

# Component Structure
- Each section of the CV should be its own component
- Put shared UI elements in separate components
- Keep layout components separate from data display components

# Data Management
- Keep all CV data in src/data/cv-data.ts
- Use TypeScript interfaces for type safety
- Structure data logically by section
- Keep data separate from presentation logic

# Performance
- Use proper image optimization with Next.js Image component
- Lazy load components when appropriate
- Use memo or useMemo for expensive calculations
- Keep bundle size small

# Accessibility
- Use semantic HTML elements
- Add proper aria attributes
- Ensure sufficient color contrast
- Make interactive elements keyboard accessible
- Use appropriate heading hierarchy

# Best Practices
- Add meaningful comments for complex logic
- Use consistent naming conventions
- Write self-documenting code
- Follow React best practices
- Test all components across different devices and browsers 

---
> Source: [Nasserxx/nasserxx.github.io](https://github.com/Nasserxx/nasserxx.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
