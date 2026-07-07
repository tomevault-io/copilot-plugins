## staffreview

> You are an expert in Vue 3+ with Typescript,Rspack, Rsbuild, Geldata and web application development. You write maintainable, performant, and accessible code.

# AGENTS.md

You are an expert in Vue 3+ with Typescript,Rspack, Rsbuild, Geldata and web application development. You write maintainable, performant, and accessible code.

## Commands

- `pnpm run dev` - Start the dev server
- `pnpm run build` - Build the app for production
- `pnpm run preview` - Preview the production build locally

## Docs

- Rsbuild: https://rsbuild.rs/llms.txt
- Rspack: https://rspack.rs/llms.txt

## Tools

### Biome

- Run `pnpm run lint` to lint your code
- Run `pnpm run format` to format your code

## Code Style and Structure
### Code Style
- Write concise, technical TypeScript code with accurate examples.
- Use composition API and declarative programming patterns; avoid options API.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Structure files: exported component, composables, helpers, static content, types.

### Naming Conventions
- Use lowercase with dashes for directories (e.g., components/auth-wizard).
- Use PascalCase for component names (e.g., AuthWizard.vue).
- Use camelCase for composables (e.g., useAuthState.ts).

### TypeScript Usage
- Use TypeScript for all code; prefer types over interfaces.
- Avoid enums; use const objects instead.
- Use Vue 3+ with TypeScript, leveraging defineComponent and PropType.

### UI and Styling
- Use Ark UI Vue, and Tailwind for components and styling.
- Implement responsive design with Tailwind CSS; use a mobile-first approach.

### Performance Optimization
- Leverage Vue's built-in performance optimizations.
- Use Suspense for asynchronous components.
- Implement lazy loading for routes and components.
- Optimize images: use WebP format, include size data, implement lazy loading.

### Key Conventions
- Use VueUse for common composables and utility functions.
- Use Pinia for state management.
- Optimize Web Vitals (LCP, CLS, FID).
- Utilize vue's auto-imports feature for components and composables.
-  Use Axios for Promise-based HTTP client

### vue-specific Guidelines
- Follow vue 3 directory structure (e.g., pages/, components/, composables/).
- Use vue's built-in features:
- Auto-imports for components and composables.
- File-based routing in the pages/ directory.
- Server routes in the server/ directory.
- Leverage vue plugins for global functionality.
- Use useFetch and useAsyncData for data fetching.
- Implement SEO best practices using vue's useHead and useSeoMeta.

### Vue 3 and Composition API Best Practices
- Use <script setup> syntax for concise component definitions.
- Leverage ref, reactive, and computed for reactive state management.
- Use provide/inject for dependency injection when appropriate.
- Implement custom composables for reusable logic.

### Tips for Better Results
1. **Search multiple times** - Different keywords reveal different insights
2. **Combine domains** - Style + Typography + Color = Complete design system
3. **Always check UX** - Search "animation", "z-index", "accessibility" for common issues
4. **Use stack flag** - Get implementation-specific best practices
5. **Iterate** - If first search doesn't match, try different keywords

Follow the official Vue.js, Geldata documentation for up-to-date best practices on Data Fetching, Rendering, and Routing.

---
> Source: [duksosleepy/staffreview](https://github.com/duksosleepy/staffreview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
