## typesafe-ui

> These instructions apply to this repository and its descendants unless a more specific AGENTS.md applies. Read README.md, the relevant workspace manifest, and the code you intend to change before editing.

# Agent guide — TypeSafe UI

These instructions apply to this repository and its descendants unless a more specific AGENTS.md applies. Read README.md, the relevant workspace manifest, and the code you intend to change before editing.

## Purpose and boundaries

This is an independent community UI workspace for TypeSafe AI projects. It is not the official TypeSafe SDK, a model implementation, or a published component package. Keep mock/preview states distinct from live provider results. Do not introduce API calls, credentials, telemetry, or production integrations as an incidental documentation or styling change.

## Workspace and package manager

Use pnpm, with the exact version in the root `packageManager` field. Keep `pnpm-lock.yaml` as the sole dependency lockfile and preserve `workspace:` dependencies. Install with `pnpm install --frozen-lockfile`; do not switch package managers or upgrade dependencies to complete unrelated work.

- `apps/web`: Next.js site, component catalog, demos, and Lab.
- `packages/ui`: reusable components, hooks, utilities, and design tokens.
- `packages/eslint-config` and `packages/typescript-config`: shared tooling.
- `apps/web/lib/registry.ts`: registry ids, groups, source/export paths, and states.
- `apps/web/components/demos.tsx`: demonstrations keyed by registry id.
- `apps/web/lib/site.ts`: site identity, links, language, and direction.

Resolve the installed Next.js documentation from the web workspace if the package is not available at the repository root. Preserve the generated Next.js block below.

## Implementation rules

Keep reusable primitives in `packages/ui`, not duplicated inside the demo app. Preserve the public workspace export paths in `packages/ui/package.json`. When adding a component, update the registry, demo, source display, and import example together.

Use Base UI composition (`render`, and `nativeButton={false}` for link buttons) rather than assuming Radix `asChild` semantics. Follow the existing source style and formatter configuration. Prefer shared CSS variables over one-off hardcoded colors; preserve TypeSafe-inspired pink/teal styling, both themes, logical properties, and RTL support.

Preserve semantic controls, accessible names, visible focus, keyboard navigation, and readable contrast. Do not make status intelligible by color alone. Keep server-side source loading separate from client interaction, and never put provider secrets in client components or `NEXT_PUBLIC_` variables.

## Verification

Run from the repository root:

```sh
pnpm lint
pnpm typecheck
pnpm build
pnpm test:e2e
node --test scripts/social-metadata.test.mjs
```

For UI changes, also inspect the component's Preview and Source tabs, Install and Import examples, `/lab`, keyboard behavior, light/dark modes, narrow screens, and RTL. `pnpm format` writes files; avoid unrelated formatting churn.

There is no root unit-test script named `test`; the root does define `test:e2e`. Do not invent `pnpm test`, report manual inspection as automated coverage, or claim a build proves accessibility. The standalone social-metadata check validates source invariants, not a rendered HTTP response. State which checks ran, their results, and any environment limitation.

## Documentation and handoff

Keep README.md and CONTRIBUTING.md consistent with the manifests. Keep CLAUDE.md as a pointer to this file rather than a competing policy. Preserve attribution to shadcn/ui, Base UI, TypeSafe-inspired branding, and the OpenCoven layout inspiration; do not imply vendor endorsement.

Read `docs/discovery/README.md` for sharing and screenshot guidance. The generated OG route must use public editorial copy only, never request content, credentials, or live model calls. Preserve existing per-page metadata and do not canonicalize all routes to the homepage.

`repository-metadata.json` records intended GitHub About text/topics only. Applying it requires a separate authorized GitHub settings action; editing the file alone does not publish topics. Do not create release tags, publish packages, change licensing, or change repository visibility unless explicitly requested.

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

---
> Source: [TypeSafeAI/typesafe-ui](https://github.com/TypeSafeAI/typesafe-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
