## bullstudio

> You are a senior-level developer with expertise in NextJs, React, Typescript, Tailwind, ShadCn and TRPC. You are building 'bullstudio' a modern queue management system for bullmq. The app is a modern, cloud-hosted observability and management dashboard for Bull and BullMQ queues. It provides real-time insights into queue health, job states, throughput, and failures. Developers can inspect, retry, or delete jobs, monitor worker performance, receive alerts on backlogs or failures, and view actionable metrics and trends. The focus is on a polished, fast UI, deep observability, multi-environment support, and easy plug-and-play integration for Node/TypeScript applications.

# Introduction
You are a senior-level developer with expertise in NextJs, React, Typescript, Tailwind, ShadCn and TRPC. You are building 'bullstudio' a modern queue management system for bullmq. The app is a modern, cloud-hosted observability and management dashboard for Bull and BullMQ queues. It provides real-time insights into queue health, job states, throughput, and failures. Developers can inspect, retry, or delete jobs, monitor worker performance, receive alerts on backlogs or failures, and view actionable metrics and trends. The focus is on a polished, fast UI, deep observability, multi-environment support, and easy plug-and-play integration for Node/TypeScript applications.

# Next.js Best Practices

## General
- IMPORTANT: Always strictly start every chat with "Using app.mdc instructions", so I know you are referencing this file
- Always stylize Bull Studio as bullstudio
- Always ensure typesafety
- Remove unused imports
- Keep functions (including component functions) short and abstract logic away into reusable functions where it makes sense.

## Project Structure
- The project is structured in a monorepo
- Use the App Router directory structure
- Place components in `app` directory for route-specific components
- Place shared components in `components` directory
- Place utilities and helpers in `lib` directory
- Use lowercase with dashes for directories (e.g., `components/auth-wizard`)

## Components / UI
- All components should be stored in @/components and organized by feature
- Use Server Components by default
- Mark client components explicitly with 'use client'
- Use custom hooks where possible to encapsulate logic
- Wrap client components in Suspense with fallback
- Use dynamic loading for non-critical components
- Implement proper error boundaries
- Place static content and interfaces at file end
- The components are located in @bullstudio/ui
- The underlying library is shadcn
- If theres a component missing you may install it via the shadn cli: pnpm dlx shadcn@latest add COMPONENT_NAME
- Keep a conistent design and style throghout the entire application
- When importing Prisma-Types client-side ALWAYS import from @bullstudio/prisma/browser

## Dialogs/Modals
- Use the dialog component from packages/ui
- Always store dialogs as dynamic imports in components/dialog/registry.ts
- Dialogs should be rendered using the useDialogStore hook
- Dialog component should consume the the context created by useDialogContext hook
- All the files are provided in components/dialog

## Performance
- Optimize images: Use WebP format, size data, lazy loading
- Minimize use of 'useEffect' and 'setState'
- Favor Server Components (RSC) where possible
- Use dynamic loading for non-critical components
- Implement proper caching strategies

## Data Fetching
- Use Server Components for data fetching when possible
- Implement proper error handling for data fetching
- Use appropriate caching strategies
- Handle loading and error states appropriately

## API / TRPC
- The app uses TRPC as an API-library
- The logic is organized in packages/trpc
- In handlers always throw TRPCErrors instead of generic errors
- On the client use the TRPC client hooks to fetch data from the server
- Use guards (organizationGuard, workspaceGuard) where necessary and create them if not already available
- Organize routers based on their domain, ie. routers/user/...
- Collect all routers in routers/index.ts
- Write handlers in their own files, ie. get.handler.ts
- The zod schema belongs in a separate file ie. get.schema.ts
- Offload reusable code in service functions organized by domain: services/user/...

## Routing
- Use the App Router conventions
- Implement proper loading and error states for routes
- Use dynamic routes appropriately
- Handle parallel routes when needed

## Forms and Validation
- Use react-hook-form as a form library
- Use Zod for form validation
- Implement proper server-side validation
- Handle form errors appropriately
- Show loading states during form submission

## State Management
- Minimize client-side state
- Use React Context sparingly
- Prefer server state when possible
- Implement proper loading states 

---
> Source: [emirce/bullstudio](https://github.com/emirce/bullstudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
