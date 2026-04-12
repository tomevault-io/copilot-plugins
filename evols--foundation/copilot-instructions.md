## foundation

> For application_frontend

call the backend with tanstack query. use use_query_wrapper ( application_frontend/src/hooks/queries/toast_utils.ts ) which relies on tanstack's useSuspenseQuery, which uses suspense
use suspense whenever possible
tanstack query hooks are in application_frontend/src/hooks/queries

pages are in application_frontend/src/routes

wrap error boundaries in HeadlessBoundary ( application_backend/src/components/reusable/headless_boundary.tsx )
if you need to throw an exception, throw an HeadlessError ( application_frontend/src/components/reusable/headless_error.ts ) because it can be rendered by HeadlessBoundary

try to use reusable components (in application_backend/src/components/reusable ) or shadcn-generated components (in application_backend/src/components/ui ). you can try to generate new components using shadcn-ui -> pnpm dlx shadcn@latest add {component_name}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Evols)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Evols)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
