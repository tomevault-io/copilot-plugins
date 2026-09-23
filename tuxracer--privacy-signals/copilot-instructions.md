## privacy-signals

> A zero-dependency npm library that detects browser privacy opt-out signals: legacy Do Not Track and Global Privacy Control. Two exports: `isTrackingOptedOut()` (combined boolean) and `getPrivacySignals()` (per-signal breakdown, `PrivacySignals` type). ESM-only, TypeScript, published to npm as `privacy-signals`.

# privacy-signals

A zero-dependency npm library that detects browser privacy opt-out signals: legacy Do Not Track and Global Privacy Control. Two exports: `isTrackingOptedOut()` (combined boolean) and `getPrivacySignals()` (per-signal breakdown, `PrivacySignals` type). ESM-only, TypeScript, published to npm as `privacy-signals`.

Extracted from the dashradar app (`~/Development/dashradar-app`, `src/lib/doNotTrack`), which will consume this package.

## Constraints

- **Zero runtime dependencies.** This is a selling point of the package; do not add one. Inline small helpers instead (the remeda `isString` check was inlined during extraction for exactly this reason).
- **Safe outside the browser.** The function must return `null`, never throw, when the signal cannot be read: no context that carries it (SSR, edge runtimes, tests) or a property read that throws. Keep the `typeof` guards and the `try`/`catch`. `null` means unknown, distinct from `false` (readable, no opt-out) — never collapse the two.
- **A global `navigator` does not mean a browser.** Node 21+, Deno, Bun, and edge runtimes define a minimal one, so a navigator-only guard returns `false` on the server instead of `null`. Inside a window the signal is read directly; outside one, the guard requires a worker scope _and_ that `navigator` actually carries a signal property. That property test is the part that matters: Cloudflare Workers presents a `ServiceWorkerGlobalScope` but gives `navigator` only `userAgent`/`sendBeacon`, so a scope-shape check alone would let an edge server pass as a browser. Do not "simplify" this to a single `typeof` guard.
- **Workers are supported on purpose.** The GPC spec exposes `globalPrivacyControl` on `WorkerNavigator`, so workers return a real `true`/`false`. Do not narrow the guard back to requiring `window`.
- **No global type augmentation.** The non-standard properties (`doNotTrack`, `msDoNotTrack`, `globalPrivacyControl`) are read via the `readProperty` helper (`Reflect.get` returning `unknown`), not by augmenting `Navigator`/`Window` or using `as` assertions. Augmentations would leak into consumers' type space.
- **ESM-only.** No CJS build, no dual-package setup. The `exports` map in `package.json` is the whole public surface.

## Commands

```bash
pnpm test        # Run tests once (vitest run, jsdom)
pnpm test:watch  # Run tests in watch mode
pnpm build       # tsc -p tsconfig.build.json → dist/
pnpm check       # Verify formatting + typecheck (run before commits)
pnpm format      # Auto-fix formatting (prettier --write)
```

**Important**: Always run `pnpm check` before commits. It only verifies formatting; run `pnpm format` to fix.

## Publishing

`pnpm publish` runs the `prepublishOnly` gate (check + test + build) automatically. Only `dist/` ships (`files` in `package.json`); `tsconfig.build.json` excludes `src/tests.ts` from the build.

## Gotchas

- **pnpm 11 blocks dependency build scripts by default.** `pnpm-workspace.yaml` allowlists esbuild's postinstall (`allowBuilds`); a fresh clone errors on `pnpm install` without it. If a new dev dependency needs a build script, add it there.
- **Tests need jsdom** (configured in `vitest.config.ts`) because they define properties on `navigator`/`window`. Tests must restore the globals they touch; use the existing `withProperty` helper, which returns a cleanup.

## Coding Standards

- **Package manager**: `pnpm` for everything.
- **ESM imports only**: `import`, never `require()`.
- **Arrow functions**: `const foo = () => { ... }`.
- **Named constants**, no magic numbers; underscore separators for numbers 1000 and up (`1_500`).
- **Sidecar files as the module grows**: exported types live in the `types.ts` sidecar (`PrivacySignals` is there now); `index.ts` re-exports them. If the module ever exports constants, move them all to a `consts.ts` sidecar the same way (never export constants or types directly from `index.ts`).
- **Type guards over `as` assertions** on runtime-unknown values.
- **JSDoc**: skip `@param`/`@returns`; types cover them.
- **Tests verify behavior, not implementation**: test what the function returns for each browser signal, never internals.
- **No em dashes or AI-isms in docs**: plain, direct voice; no "delve", "seamless", "robust", "leverage", adjective triads, emoji headings, or "In summary" wrap-ups.

---
> Source: [tuxracer/privacy-signals](https://github.com/tuxracer/privacy-signals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
