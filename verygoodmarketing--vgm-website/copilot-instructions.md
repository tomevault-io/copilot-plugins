## vgm-website

> - `npm run dev` - Start development server

# VGM Website Development Guide

## Commands

- `npm run dev` - Start development server
- `npm run build` - Build production version
- `npm run preview` - Preview production build
- `npm run format` - Format code with Prettier
- `npm run lint` - Check code formatting with Prettier
- `npm run typecheck` - Type check TypeScript files
- `npm start` - Alias for `npm run dev`

## Style Guidelines

- **Imports**: Sort imports: 1) React/libs 2) Components 3) Utils/local imports. Use `@/` path aliases.
- **TypeScript**: Use strict type checking. Prefer explicit types over `any`. Component props should be typed.
- **Components**: React components use `.tsx`, Astro components use `.astro`. React state/props for local state.
- **Naming**: PascalCase for components, camelCase for variables/functions. Clear, descriptive names.
- **CSS**: Use Tailwind utility classes with composition via `cn()` utility from `/lib/utils.ts`.
- **Formatting**: 100 character line limit, 2 space indentation. Single quotes for strings.
- **Error handling**: Use try/catch with specific error types. Log errors appropriately.
- **File organization**: Group by feature in the `src` directory. Modular component architecture.
- **Testing**: Add data-testid attributes to components that need to be tested.
- **Commits**: Clear, concise commit messages describing changes. Use imperative mood.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/verygoodmarketing)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/verygoodmarketing)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
