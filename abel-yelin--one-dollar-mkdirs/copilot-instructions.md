## one-dollar-mkdirs

> This document outlines the structure of the Next.js application and key entry points.

# Application Structure

This document outlines the structure of the Next.js application and key entry points.

## App Router Structure

The application uses Next.js App Router with a split between website and Sanity Studio routes:

- **src/app/(website)**: Public-facing website routes
- **src/app/(sanity)**: Sanity Studio admin routes
- **src/app/api**: API routes for various functionalities

## Key Routes

- **/**: Main landing page
- **/auth**: Authentication-related pages (sign-in, sign-up)
- **/[item|category|tag|collection]**: Directory listings pages
- **/blog**: Blog section
- **/dashboard**: User dashboard
- **/studio**: Sanity Studio CMS interface

## Authentication

Authentication is handled through NextAuth with various providers and credentials authentication:

- **src/auth.ts**: Main authentication configuration
- **src/auth.config.ts**: Auth providers and callbacks

## Components Organization

Components are organized into several categories:

- **UI Components**: Basic reusable UI elements
- **Layout Components**: Page layouts and structural components
- **Feature Components**: Components specific to certain features
- **Form Components**: Input and form-related components

## Data Flow

The application follows these data flow patterns:

1. **Server Components**: Fetch data directly from the database/CMS
2. **Server Actions**: Handle data mutations and form submissions
3. **API Routes**: Provide data endpoints for client components
4. **Client Components**: Use hooks for client-side data fetching and state management

## Middleware

- **src/middleware.ts**: Handles authentication, redirects, and other middleware functionality
- **src/routes.ts**: Defines route configurations and access control

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abel-yelin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
