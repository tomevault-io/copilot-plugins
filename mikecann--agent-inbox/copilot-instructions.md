## type-route-rules

> See https://type-route.zilch.dev/ for more

# Type Route Guidelines

See https://type-route.zilch.dev/ for more

## Basic Setup

### Route Definition
- ALWAYS use `defineRoute` to define routes with proper parameter types:
```typescript
import { createRouter, defineRoute, param } from "type-route";

export const { RouteProvider, useRoute, routes } = createRouter({
  home: defineRoute("/"),
  detail: defineRoute(
    {
      id: param.path.string,
      tab: param.query.optional.string,
    },
    (p) => `/detail/${p.id}`
  ),
});
```

### Parameter Types
- Use `param.path` for URL path parameters
- Use `param.query` for query string parameters
- Use `param.query.optional` for optional query parameters
- Available parameter types:
  - `string`
  - `number`
  - `boolean`
  - `array`

### Route Provider Setup
- Wrap your app with `RouteProvider`:
```typescript
import { RouteProvider } from "./routes";

function App() {
  return (
    <RouteProvider>
      <YourApp />
    </RouteProvider>
  );
}
```

## Navigation

### Link Navigation
- Use the `link` property for anchor tags:
```typescript
<a {...routes.detail({ id: "123" }).link}>
  View Details
</a>
```

### Programmatic Navigation
- Use `push()` for regular navigation:
```typescript
routes.detail({ id: "123" }).push();
```
- Use `replace()` to replace current history entry:
```typescript
routes.detail({ id: "123" }).replace();
```

## Route Helpers

### Current Route Access
- Use `useRoute()` to get current route info:
```typescript
const route = useRoute();
if (route.name === "detail") {
  console.log(route.params.id);
}
```

### Type Safety
- Route parameters are fully typed
- TypeScript will catch invalid parameter types
- Route names are type-safe in switch statements

## Best Practices

### Route Organization
- Keep all routes in a single `routes.ts` file
- Export helper hooks for common route operations
- Example helper hook:
```typescript
export function useCurrentId(): string | undefined {
  const route = useRoute();
  if (route.name === "detail")
    return route.params.id;
  return undefined;
}
```

### Route Parameters
- Use path parameters for required values
- Use query parameters for optional values
- Keep parameter names descriptive and consistent

### Type Safety
- Let TypeScript infer route parameter types
- Don't use type assertions unless absolutely necessary
- Validate parameters before using them in components

### Navigation
- Prefer declarative navigation with `link` over imperative with `push`
- Handle navigation errors gracefully
- Consider using route helpers for complex navigation logic

## Common Patterns

### Optional Parameters
```typescript
const route = defineRoute(
  {
    id: param.path.string,
    view: param.query.optional.string,
  },
  (p) => `/item/${p.id}`
);
```

### Nested Routes
```typescript
const routes = {
  users: defineRoute("/users"),
  userDetail: defineRoute(
    { id: param.path.string },
    (p) => `/users/${p.id}`
  ),
  userSettings: defineRoute(
    { id: param.path.string },
    (p) => `/users/${p.id}/settings`
  ),
};
```

### Route Groups
```typescript
const routes = {
  home: defineRoute("/"),
  auth: {
    login: defineRoute("/auth/login"),
    register: defineRoute("/auth/register"),
    forgot: defineRoute("/auth/forgot"),
  },
};
``` 

---
> Source: [mikecann/agent-inbox](https://github.com/mikecann/agent-inbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
