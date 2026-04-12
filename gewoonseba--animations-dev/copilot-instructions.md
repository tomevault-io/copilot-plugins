## animations-dev

> - This is a Next.js 15 project with React 19 and TypeScript


# Development Workflow

## Project Setup

- This is a Next.js 15 project with React 19 and TypeScript
- Uses Tailwind CSS 4 for styling
- Development server runs with Turbopack: `npm run dev`

## Adding New Exercises

See [Exercise Creation Workflow Rules](./exercise-creation.mdc) for detailed instructions on adding new exercises, including file structure, TypeScript and CSS Modules conversion, and project conventions.

## Code Quality

- Use TypeScript strict mode (enabled in [tsconfig.json](mdc:tsconfig.json))
- Follow the established component patterns
- Keep components focused and single-purpose
- Use semantic HTML elements
- Ensure animations are accessible (respect `prefers-reduced-motion`)
- Always lint and format code you've created using `pnpm run lint` and `pnpm run format`

## File Naming Conventions

- Use kebab-case for directory names: `card-hover`, `text-reveal`
- Use kabab-case for file names: `exercise-link.tsx`
- Use PascalCase for component names: `CardHoverComponent`
- Use camelCase for CSS class names in modules
- Use descriptive names that explain the animation behavior

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gewoonseba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
