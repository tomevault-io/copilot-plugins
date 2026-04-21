## express-server-typescript-template

> how to create a server


- to create a server we use 2 files: `initServer.ts` and `buildApp.ts`.
- `initServer.ts` is responsible for:
  - calling `buildApp()` to get the composed app.
  - read the port from config.
  - start the HTTP server listening.
  - register global handlers for `unhandledRejection` and `uncaughtException`.
- `buildApp.ts`'s job is to expose an async function called `buildApp`.
- `buildApp` should:
  - create the raw Express app.
  - instantiate `AppFactory` with the `app` and `optimizedApp`.
  - register plugins, modules, error handler, and path-not-found handler.
  - modules are attached **after** plugins, so they can use plugin-attached services.
  - return the composed app.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/talkohavy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
