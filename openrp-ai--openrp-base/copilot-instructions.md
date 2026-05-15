## api-routes

> how to write api routes

- our api routes are under `app/api`
- like our pages, api routes use kebab-casing
- our apis are RESTful and should be as simple as possible
- use zod schema to parse requests
- use `parseSearchParams` from `zod-search-params` to parse search params
- api routes contain as little logic as possible and act as request/response parser. The main data logic should be under `lib/api`
- it should only contain routes. ALL other utilities and core business logic and even helper functions should be in `lib/`
- use `createSuccessResponse` and `createErrorResponse` in @api_response.ts and a try block, like so:
```
try {
  return createSuccessResponse(await someApi(parsedParams));
} catch (error) {
  return createErrorResponse(error);

}

```

---
> Source: [openrp-ai/openrp-base](https://github.com/openrp-ai/openrp-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
