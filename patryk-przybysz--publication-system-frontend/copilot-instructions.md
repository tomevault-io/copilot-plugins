## publication-system-frontend

> UI component and styling guidelines using Shadcn UI, Radix UI, and Tailwind

# UI Components and Styling

## UI Framework
- Use Shadcn UI and Tailwind for components and styling
- Implement responsive design with Tailwind CSS using a mobile-first approach

## Install new Shadcn components

```sh
bunx shadcn@latest add COMPONENT
```

Example:

```sh
bunx shadcn@latest add progress
```

## Data Fetching with Tanstack Query
For API get requests to server use the `tanstack@react-query` package:

```typescript
const { data, isLoading, error } = useQuery({ ... })
```
Prefer using `useSuspenseQuery` and Tanstack Router's route-level `pendingComponent` where applicable.

## Avoid large components with nested rendering functions

Do not add multiple rendering functions inside your application, this gets out of control pretty quickly. What you should do instead is if there is a piece of UI that can be considered as a unit, is to extract it in a separate component.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/patryk-przybysz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
