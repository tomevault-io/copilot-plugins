## tanstack-router-strello

> We use Sentry for watching for errors in our deployed application, as well as for instrumentation of our application.

We use Sentry for watching for errors in our deployed application, as well as for instrumentation of our application.

## Error collection

Error collection is automatic and configured in `src/router.tsx`.

## Instrumentation

We want our server functions intstrumented. So if you see a function name like `createServerFn`, you can instrument it with Sentry. You'll need to import `Sentry`:

```tsx
import * as Sentry from '@sentry/browser'
```

And then wrap the implementation of the server function with `Sentry.startSpan`, liks so:

```tsx
Sentry.startSpan({ name: 'Requesting all the pokemon' }, async () => {
  // Some lengthy operation here
  await fetch('https://api.pokemon.com/data/')
})
```

---
> Source: [devagrawal09/tanstack-router-strello](https://github.com/devagrawal09/tanstack-router-strello) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
