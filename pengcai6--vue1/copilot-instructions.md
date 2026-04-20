## vue1

> You are an expert in TypeScript, Node.js, Vue 3, Element Plus, VueUse, and Tailwind.

You are an expert in TypeScript, Node.js, Vue 3, Element Plus, VueUse, and Tailwind.

Code Style and Structure

- Write concise, technical TypeScript code with accurate examples.
- Use composition API and declarative programming patterns; avoid options API.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Structure files: exported component, composables, helpers, static content, types.

Naming Conventions

- Use lowercase with dashes for directories (e.g., components/auth-wizard).
- Use PascalCase for component names (e.g., AuthWizard.vue).
- Use camelCase for composables (e.g., useAuthState.ts).

TypeScript Usage

- Use TypeScript for all code; prefer types over interfaces.
- Avoid enums; use const objects instead.
- Use Vue 3 with TypeScript, leveraging defineComponent and PropType.

Syntax and Formatting

- Use arrow functions for methods and computed properties.
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
- Use template syntax for declarative rendering.

UI and Styling

- Use Element Plus and Tailwind CSS for components and styling.
- Implement responsive design with Tailwind CSS; use a mobile-first approach.

Performance Optimization

- Leverage Vite's built-in performance optimizations.
- Use Suspense for asynchronous components.
- Implement lazy loading for routes and components.
- Optimize images: use WebP format, include size data, implement lazy loading.

Key Conventions

- Use VueUse for common composables and utility functions.
- Use Pinia for state management.
- Optimize Web Vitals (LCP, CLS, FID).
- Utilize unplugin-vue-components for auto-importing Element Plus components.
- Utilize unplugin-vue-router for file-based routing.

Vite-specific Guidelines

- Follow Vue 3 + Vite directory structure (e.g., src/pages/, src/components/, src/composables/).
- Use unplugin-vue-router for automatic route generation from file structure.
- Use unplugin-vue-components for auto-importing components.
- Configure Vite aliases: @ and ~/ pointing to src/ directory.
- Leverage Vite's esbuild for fast builds.

Element Plus Guidelines

- Use Element Plus components for UI building blocks.
- Configure auto-import using unplugin-vue-components with ElementPlusResolver.
- Use Element Plus icons from @element-plus/icons-vue.
- Customize theme using SCSS variables in styles/element/.
- Import specific component styles when using on-demand imports.

Vue 3 and Composition API Best Practices

- Use <script setup> syntax for concise component definitions.
- Leverage ref, reactive, and computed for reactive state management.
- Use provide/inject for dependency injection when appropriate.
- Implement custom composables for reusable logic.

Follow the official Vue.js and Element Plus documentation for up-to-date best practices on Data Fetching, Rendering, and Routing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pengcai6) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
