## zen-iptv-web

> This is a SvelteKit web application for Zen IPTV. It serves as the website for the service, handling user accounts, subscriptions, and payments.

# Project Overview

This is a SvelteKit web application for Zen IPTV. It serves as the website for the service, handling user accounts, subscriptions, and payments.

**Key Technologies:**

- **Frontend Framework:** SvelteKit
- **Build Tool:** Vite
- **Styling:** Tailwind CSS
- **Backend/Database:** Supabase (for authentication and database interactions)
- **Payments:** Stripe
- **Internationalization:** svelte-i18n
- **Deployment Adapter:** Node.js

**Architecture:**

The project follows the SvelteKit convention of file-system based routing.

- `src/routes`: Contains the main pages and API endpoints.
  - `+layout.svelte`: Defines the main layout of the application.
  - `+page.svelte`: The home page.
  - `account/`: User account related pages.
  - `api/stripe/`: Stripe webhook endpoint.
  - `auth/`: Authentication related pages and callbacks.
  - `help/`: Help page.
- `src/lib`: Contains reusable components, utilities, and services.
  - `features/account/`: Components, models, and services related to user accounts and subscriptions (RevenueCat integration).
  - `features/landing/`: Components and sections for the landing page.
  - `i18n/`: Internationalization files (English and French).
  - `shared/components/`: Generic UI components.
- `static/`: Static assets like fonts, icons, and images.

# Building and Running

This project uses `yarn` for package management.

**Development:**

To start the development server:

```bash
yarn dev
```

To start the server and open the app in a new browser tab:

```bash
yarn dev -- --open
```

**Building for Production:**

To create a production-ready build of your app:

```bash
yarn build
```

**Previewing the Production Build:**

To preview the production build locally:

```bash
yarn preview
```

**Type Checking:**

To run SvelteKit's type checker:

```bash
yarn check
```

To run the type checker in watch mode:

```bash
yarn check:watch
```

**Linting and Formatting:**

To lint the code with Prettier and ESLint:

```bash
yarn lint
```

To format the code with Prettier:

```bash
yarn format
```

# Development Conventions

- **Code Style:** Enforced by Prettier and ESLint. Run `yarn format` to automatically format the code.
- **Type Safety:** SvelteKit's built-in type checking is used. Run `yarn check` to verify types.
- **Routing:** Follows SvelteKit's file-system based routing.
- **Component Structure:** Reusable components are located in `src/lib/shared/components` and feature-specific components are in `src/lib/features/`.
- **Internationalization:** Uses `svelte-i18n` with translation files in `src/lib/i18n/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/romaingyh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/romaingyh)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
