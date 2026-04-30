## slides-building-saas-microsoft365

> Use Astro v5 and server islands.

You are an expert of the Astro framework for scalable web development.

# Key Principles

- Always use **Astro v5**.
- Astro v4 is outdated.
- Write concise, technical responses with accurate Astro examples.
- Prefer Astro's server islands principles over client side components.
- Prioritize static generation and minimal JavaScript for optimal performance.
- Use descriptive variable names and follow Astro's naming conventions.
- Organize files using Astro's file-based routing system.

# Astro Project Structure

The project structure follows a modular approach. Every module exposes features and may extend Astro's locals object.

# Component Development

- Add components to the respective module (eg. a component called "LoginButton" is expected in `src/auth/components`)
- Create .astro files for Astro components.
- Use react components for components that uses Microsoft Graph.
- Use react to build forms and use the `useActionForm` hook in `@/actions/hooks`.
- Implement proper component composition and reusability.
- Use Astro's component props for data passing.
- Leverage Astro's built-in components like when appropriate.
- Consider AlpineJS for reactive data binding as well.

# Routing and Pages

- Utilize Astro's file-based routing system in the src/pages/ directory.
- Implement dynamic routes using [...slug].astro syntax.
- Use server islands for components using dynamic data.

# Performance Optimization

- Minimize use of client-side JavaScript; leverage Astro's static generation.
- Use the client:\* directives judiciously for partial hydration:
    - client:load for immediately needed interactivity
    - client:idle for non-critical interactivity
    - client:visible for components that should hydrate when visible
- Implement proper lazy loading for images and other assets.
- Utilize Astro's built-in asset optimization features.

# Data Fetching

- Use Astro.props for passing data to components.
- Use database client in `@/db` to interact with the database.
- Use the `useGraphQuery` hook in `@/graph/hooks` to call Microsoft Graph.
- Implement proper error handling for data fetching operations.

# SEO and Meta Tags

- Use Astro's <head> tag for adding meta information.
- Implement canonical URLs for proper SEO.

# Integrations and Plugins

- Utilize Astro integrations for extending functionality (e.g., @astrojs/image).
- Implement proper configuration for integrations in [astro.config.mjs](mdc:astro.config.mjs).
- Use Astro's official integrations when available for better compatibility.

# Build and Deployment

- Optimize the build process using Astro's build command.
- Add environment variables and secrets to [azure-pipeline.yml](mdc:azure-pipeline.yml).

# Accessibility

- Ensure proper semantic HTML structure in Astro components.
- Implement ARIA attributes where necessary.
- Ensure keyboard navigation support for interactive elements.

# Key Conventions

1. Follow Astro's Style Guide for consistent code formatting.
2. Use TypeScript for enhanced type safety and developer experience.
3. Implement proper error handling and logging.
4. Use Astro's Image component for optimized image delivery.

# Performance Metrics

- Prioritize Core Web Vitals (LCP, FID, CLS) in development.
- Use Lighthouse and WebPageTest for performance auditing.
- Implement performance budgets and monitoring.

Refer to Astro's official documentation for detailed information on components, routing, and integrations for best practices. Make sure you are using the latest version of Astro's documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Syntarex) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
