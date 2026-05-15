## architecture

> - NextJS App Router for frontend and backend

# Major frameworks
- NextJS App Router for frontend and backend
- Supabase for database and authentication
- Drizzle ORM with queries for database fetching, database definition is in
  `database/`
- we are transitioning from Supabase Data API to Drizzle, so there are still
  some Supabase data fetching
- tailwindcss for styling
- shadcn for ui components (try not to modify or introduce new components under
  `components/ui`, those are reserved for shadcn. Instead just add them to the
  relevant component folder for a particular page)
- protobuf-es for app-level data. Proto definitions are under `proto/` and they
  are generated to `generated/proto`. When we fetch from the database, we
  immediately transform them into proto objects
- react-hook-form for any form building
- stepperize for multi-step components
- swr for any client-side data fetching or submission, with a universal fetcher
  in [fetchers.ts](mdc:lib/swr/fetchers.ts)
- stripe for payment
- immer and useImmer instead of React.useState for modifying complex data states
- lucide-react for icons
- nuqs for manipulating query parameters
- git for tracking user's worlds (think of them as repos). We keep a copy of
  user's information in the database but write to both db and git
- sonner for client-side notifications

# Key directories
- app: public-facing web pages that (mostly) map directly to the URL
- app/api: public-facing server APIs, currently only consumed by the frontend in client components
- components: reusable React components loosely organized according to the page they are mainly used in
- components/ui: Shadcn components, don't add new components to this directory
- database: Drizzle database schema definitions
- lib: models/helper functions shared by both server and client components, e.g. database operations that are common for both client APIs and server components are stored under lib/api

---
> Source: [openrp-ai/openrp-base](https://github.com/openrp-ai/openrp-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
