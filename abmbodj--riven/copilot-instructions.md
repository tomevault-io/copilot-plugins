## riven

> look for useful skills to use eveytime.

look for useful skills to use eveytime.
Edge functions: Only if you change code in supabase/functions/. You'd redeploy the specific function with npx supabase functions deploy <function-name>.

Sentry: The Vite client and Node `server/` use their own DSNs. **Supabase Edge Functions** use `npm:@sentry/deno` via [`supabase/functions/_shared/sentry.ts`](supabase/functions/_shared/sentry.ts): set the `SENTRY_DSN` secret (optional `SENTRY_ENVIRONMENT`, `SENTRY_RELEASE`, `SENTRY_TRACES_SAMPLE_RATE`) in the Supabase project, then call `await reportEdgeException(err, { request, functionName: 'your-fn' })` from `catch` blocks (see `stripe-webhook`). Prefer `withScope`-style reporting per [Supabase’s Sentry guide](https://supabase.com/docs/guides/functions/examples/sentry-monitoring) because the Deno runtime may reuse isolates between requests.

---
> Source: [abmbodj/Riven](https://github.com/abmbodj/Riven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
