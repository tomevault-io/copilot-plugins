## ship-fast-ts

> This is a Next.js 15+ TypeScript SaaS boilerplate with Stripe payments, NextAuth authentication, MongoDB database, and modern UI components using DaisyUI/TailwindCSS.

# ShipFast TypeScript SaaS Boilerplate - Cursor Rules

## Project Overview
This is a Next.js 15+ TypeScript SaaS boilerplate with Stripe payments, NextAuth authentication, MongoDB database, and modern UI components using DaisyUI/TailwindCSS.

## Tech Stack & Dependencies
- **Framework**: Next.js 15+ with App Router
- **Language**: TypeScript 5.9+
- **Database**: MongoDB with Mongoose ODM
- **Authentication**: NextAuth v5 (beta) with Google OAuth and Email providers
- **Payments**: Stripe integration with webhooks
- **Styling**: TailwindCSS 4.1+ with DaisyUI 5.0+
- **Email**: Resend for transactional emails
- **UI Components**: Headless UI, React Hot Toast, React Tooltip
- **Blog**: MDX support for blog content

## Code Style & Conventions

### TypeScript
- Use strict TypeScript with proper typing
- Prefer interface over type for object definitions
- Use proper generic types for reusable components
- Import types using `import type { }` when possible
- Use Zod for runtime validation when needed

### File Naming & Structure
- Use kebab-case for file names: `button-checkout.tsx`
- Use PascalCase for component names: `ButtonCheckout`
- Use camelCase for functions and variables
- API routes: `/app/api/[feature]/route.ts`
- Components: `/components/ComponentName.tsx`
- Pages: `/app/[route]/page.tsx`
- Types: `/types/[feature].ts`
- Utils/Libs: `/libs/[utility].ts`
- Models: `/models/[ModelName].ts`

### Component Architecture
- Use functional components with hooks
- Prefer server components by default, use "use client" only when necessary
- Export default for main component, named exports for utilities
- Keep components focused and single-responsibility
- Use proper TypeScript interfaces for props

### Import Order
1. React and Next.js imports
2. Third-party libraries
3. Internal components (components/)
4. Internal utilities (libs/)
5. Internal types (types/)
6. Config and constants
7. Relative imports

### API Routes
- Use proper HTTP methods (GET, POST, PUT, DELETE)
- Always validate request bodies
- Use proper error handling with try-catch
- Return consistent JSON responses with proper status codes
- Connect to MongoDB before database operations
- Use NextAuth session for authentication when needed

### Database & Models
- Use Mongoose for MongoDB operations
- Define proper schemas with TypeScript interfaces
- Use proper error handling for database operations
- Always connect to MongoDB before operations using `connectMongo()`
- Use proper validation in models

### Authentication
- Use NextAuth v5 patterns
- Check authentication in API routes when needed
- Use proper session handling
- Implement proper user creation and updates

### Stripe Integration
- Use proper webhook signature verification
- Handle all relevant Stripe events properly
- Update user access based on payment status
- Use proper error handling for Stripe operations
- Store customer and subscription data in user model

## Specific Patterns for This Codebase

### Config Pattern
- All configuration should go in `/config.ts`
- Use the `ConfigProps` interface from `/types/config.ts`
- Access config using `import config from "@/config"`

### Tailwind CSS v4 Configuration
- Use CSS-first configuration in `app/globals.css` with `@theme` directive
- No more `tailwind.config.js` file - all customization in CSS
- Use `@import "tailwindcss"` instead of `@tailwind` directives
- Custom theme variables defined with `--color-name`, `--spacing-name`, etc.
- PostCSS config uses `@tailwindcss/postcss` plugin

### Component Patterns
- Use DaisyUI classes for styling: `btn`, `btn-primary`, `card`, etc.
- Implement responsive design with Tailwind breakpoints
- Use proper semantic HTML elements
- Include proper accessibility attributes

### Button Components
- Follow the pattern of existing buttons like `ButtonCheckout`, `ButtonSignin`
- Use proper loading states
- Handle async operations properly
- Include proper error handling with toast notifications

### Layout Pattern
- Use the `LayoutClient` wrapper for client-side providers
- Include proper SEO meta tags using the `/libs/seo.tsx` helper
- Use proper font loading and optimization

### Blog Pattern
- Use MDX for blog content in `/app/blog/_assets/`
- Follow the existing article structure and types
- Use proper image optimization with Next.js Image component

## Environment Variables
Always use proper environment variable patterns:
- `NEXTAUTH_SECRET` for NextAuth
- `GOOGLE_ID` and `GOOGLE_SECRET` for Google OAuth
- `MONGODB_URI` for database connection
- `STRIPE_SECRET_KEY` and `STRIPE_WEBHOOK_SECRET` for payments
- `RESEND_API_KEY` for emails

## Error Handling
- Use try-catch blocks in API routes
- Log errors with descriptive messages
- Return proper HTTP status codes
- Use toast notifications for user-facing errors
- Handle loading states in components

## Performance
- Use Next.js Image component for images
- Implement proper lazy loading
- Use proper caching strategies
- Optimize bundle size with proper imports

## Security
- Validate all inputs
- Use proper CSRF protection
- Verify webhook signatures
- Sanitize user data
- Use proper authentication checks

## Testing
- Write unit tests for utility functions
- Test API routes with proper mocking
- Test components with React Testing Library
- Use proper TypeScript types in tests

## Commit Messages
Use Conventional Commits format:
- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation
- `style:` for formatting changes
- `refactor:` for code refactoring
- `test:` for adding tests
- `chore:` for maintenance tasks

## Do Not
- Don't use `any` type unless absolutely necessary
- Don't bypass TypeScript strict mode
- Don't forget to handle loading and error states
- Don't hardcode configuration values
- Don't forget to validate user inputs
- Don't skip error handling in async operations
- Don't use inline styles, prefer TailwindCSS classes
- Don't create components without proper TypeScript interfaces
- Don't forget to connect to MongoDB in API routes
- Don't skip webhook signature verification for Stripe 

---
> Source: [ayakaloveyuri/ship-fast-ts](https://github.com/ayakaloveyuri/ship-fast-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
