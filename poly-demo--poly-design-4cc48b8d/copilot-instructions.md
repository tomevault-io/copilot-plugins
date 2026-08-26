## poly-design-4cc48b8d

> React component library published as `@juristr/tusky-design`. Read this before touching code.

# Agent instructions — `poly-design`

React component library published as `@juristr/tusky-design`. Read this before touching code.

## Where to touch

- **Components**: `packages/tusky-design/src/lib/<Component>/`. One folder per component:
  - `<Component>.tsx` — the component
  - `index.ts` — barrel re-export
- **Public surface**: add new components to `packages/tusky-design/src/index.ts`.
- **Styles**: Tailwind v4 utility classes inline. Global tokens in `packages/tusky-design/src/styles.css`.

## Tooling

- npm workspaces. The single library lives at `packages/tusky-design`.
- Vite library mode → ESM only (`dist/index.js` + `dist/index.d.ts` via `vite-plugin-dts`).
- React, react-dom, lucide-react are `peerDependencies` — never bundle them.
- TS: `packages/tusky-design/tsconfig.lib.json` drives types. Typecheck is part of `build`.
- No tests, no Storybook. Don't add them.

## Conventions

- Use `clsx` for conditional classes (only runtime dep besides peers).
- Keep components presentational; no data fetching, no router coupling.
- Lib build is consumed two ways: published `@juristr/tusky-design` on npm, and tarball-packed for Polygraph demos. Keep `peerDependencies` honest.

## Verifying changes

```sh
npm install
npm run typecheck    # tsc --noEmit on the lib
npm run build        # typecheck + vite build → packages/tusky-design/dist
```

---
> Source: [Poly-Demo/poly-design-4cc48b8d](https://github.com/Poly-Demo/poly-design-4cc48b8d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
