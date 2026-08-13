## art

> quantizor's studio: personal creative-coding site on TanStack Start (file-based routes, Nitro, Bun). Ships as a static GitHub Pages tree under `docs/` via `scripts/deploy.sh`.

quantizor's studio: personal creative-coding site on TanStack Start (file-based routes, Nitro, Bun). Ships as a static GitHub Pages tree under `docs/` via `scripts/deploy.sh`.

Stack pins (installed)
- TanStack Start / Router: 1.159.4
- React: 19.2.4
- Three.js / @types/three: 0.185.x (WebGPURenderer from `three/webgpu`)
- TypeGPU: 0.9.x (compute only; unused in source until a compute shader lands)
- Tailwind CSS: 4.1.x
- Zod: 3.25.x
- TypeScript: 5.9.x, `strict: true`
- Runtime / tests: Bun

Local carve-outs (override global defaults for this repo)
- No CI. Do not add GitHub Actions or other hosted checks. Local `bun run verify` is the gate.
- No test coverage thresholds or coverage reporting. Do not add a second test runner for branch metrics. Tests accompany behavior changes and must be able to fail for the reason that matters.
- Accessibility work is deprioritized. Do not open WCAG remediation unless the user asks. Keep the `/ui` showcase in sync when design-system APIs change.

Agent directives
- Never run the dev server. Never run `bun dev`, `bun start`, `bun run preview`, or any long-running preview. Assume the app is already being served at `http://art.localhost:3011/` and reuse it. Verification is `bun run typecheck`, `bun test`, and `bun run build` (or `bun run verify`).
- Do not spawn a second Vite/Bun server beside the running one; the port is taken and a stray server splits the logs.
- When planning, include Do's and Don'ts, concrete code samples, and pointers into this file or `agent-docs/`.
- Tracker for deferred work: GitHub Issues. Do not use Changesets (package is `private: true` and unpublished).
- `docs/` is generated Pages output. Never author documentation there. Agent standards live under `agent-docs/`; research notes live under `research/`.
- lightcycle (`src/games/lightcycle/`) has no route. It mounts from `src/components/NotFound.tsx` as the 404 page and participates in the SSR graph that way.

Wayfinding
- This file is the index. Depth lives in `agent-docs/standards/*.md` and `agent-docs/{tanstack-start,react,tailwind-css,typegpu-webgpu,threejs}.md`.
- Design system living docs: `/ui` route plus `src/ui/README.md`, `src/ui/INTEGRATION.md`, `src/ui/TAILWIND-PATTERNS.md`.
- `CLAUDE.md` is `@AGENTS.md` (pointer only).

Prose and communication
- American English. No em-dashes outside quoted material. Plain language over jargon.
- Error messages say what is wrong, where, and what to do. No attribution footers on commits, PRs, or files.
- See agent-docs/standards/prose.md.

Correctness and types
- Types are law. No `any`, no non-null `!`, no `@ts-expect-error` / `@ts-ignore` except as a marked negative test.
- Prefer `unknown` plus type guards. Prefer explicit return types on exported functions.
- Model fixed value sets as `as const` objects plus derived unions, never TypeScript `enum`.
- Zod schemas validate external input and infer types. See agent-docs/standards/types.md.

Testing and validation
- Practice TDD where possible: failing test, then minimal green, then refactor.
- Ship tests with behavior changes. Cover happy paths, empty/null, edges, and failure modes.
- Prefer pure engine logic without Three.js or React. Mock DOM/Three when a component must be tested.
- Use `bun test` for the whole tree (`bun test`), not a path that silently skips suites.
- Coverage is not measured or gated here. Judge a test by whether it can fail for the reason that matters.
- See agent-docs/standards/testing.md.

Inputs and error handling
- Treat the frontend as untrusted. Validate path params, request bodies, `res.json()`, and persisted JSON with Zod or hand-written guards at the boundary.
- Server functions use `.inputValidator` with Zod. Do not use identity validators.
- Worker transferables get discriminant and `byteLength` checks, not Zod.
- See agent-docs/standards/inputs.md.

Organization and naming
- Alphabetize fields in declarations and object literals unless order is load-bearing.
- Persist identifiers in plain English. Prefer reuse of `src/ui/` and shared utils before hand-rolling.
- A source file past about a thousand lines is a prompt to ask whether to split.
- See agent-docs/standards/organization.md.

Comments and docs
- Comments explain why and non-obvious current behavior, never what changed.
- Exported members get hover-friendly docs. Settled designs are documented before implementation.
- See agent-docs/standards/comments.md.

Security and data
- Screen every user-input surface. Do not log cookies, Authorization headers, or other secrets.
- Gate debug logging behind `import.meta.env.DEV` or a tree-shakeable helper so production drops the work.
- See agent-docs/standards/security.md.

Migrations and data lifecycle
- N/A. No database.

Performance
- Validate hot-path claims with a measurement, not recall. Keep drift-prone numbers out of docs; name the command that prints the live figure.
- See agent-docs/standards/performance.md.

Git and process
- Do not commit, push, deploy, or open a PR unless the user asks. Approval is per act.
- Never `git stash`. Prefer a temporary commit when the working tree must be cleared.
- Install upgrades with lifecycle scripts off until a package proves it needs them.
- See agent-docs/standards/git.md.

Method
- Fix at the cause. No suppressions that paper over a real defect.
- One fact, one home. After renaming or changing a value, update every place that recorded the old fact in the same pass.
- Argue the opposite before calling work done. Re-read or re-run before reporting.
- See agent-docs/standards/method.md.

Aesthetic and UX
- Accessibility is deprioritized for this repo; do not open a WCAG pass unless asked.
- When `src/ui/` APIs or tokens change, update `src/routes/ui.tsx` and the adjacent UI docs in the same pass.
- Render and look at visual changes before calling them done. Prefer container queries for slot-sized layouts.
- See agent-docs/standards/aesthetic.md.

Domain and stack
- GPU rendering uses `three/webgpu` `WebGPURenderer` with TSL / NodeMaterial for materials and post-processing. Never `WebGLRenderer`, raw `ShaderMaterial`, or `EffectComposer`. Assert `renderer.backend.isWebGPUBackend === true` after `init()`. See agent-docs/threejs.md.
- TypeGPU is the authoring layer for standalone GPU compute over typed buffers. TSL owns materials and bloom graphs. Do not claim TypeGPU is practiced until a compute shader lands. See agent-docs/typegpu-webgpu.md.
- Import Three only from `three/webgpu` (and documented addons / `three/tsl`). Ban bare `three` imports in app code.
- Server functions: `.inputValidator` plus Zod. Loaders are isomorphic. See agent-docs/tanstack-start.md.
- React 19: `ref` is a prop; do not add new `forwardRef`. See agent-docs/react.md.
- Tailwind v4 via `@tailwindcss/vite`. Import UI as `~/ui`. See agent-docs/tailwind-css.md.

Project structure

```
src/
├── routes/                 # File-based routes (incl. projects, api, ui)
├── components/             # Shared shell, SpeedDial, NotFound (hosts lightcycle)
├── games/lightcycle/       # 404-mounted game (no dedicated route)
├── projects/               # id1, tension scenes and engines
├── ui/                     # Design system
├── utils/                  # Server functions, helpers
├── styles/                 # App CSS
├── router.tsx
└── routeTree.gen.ts        # Generated; do not hand-edit
agent-docs/                 # Agent standards and stack refs
research/                   # Hand-authored research notes
docs/                       # Generated GitHub Pages output only
```

Commands

```bash
bun run typecheck   # tsc --noEmit
bun test            # whole suite
bun run lint        # oxlint
bun run policy      # forbidden GPU / enum / bare-three imports
bun run verify      # typecheck + lint + policy + test + build
bun run build       # typecheck then vite build
bun run deploy      # build and prerender into docs/ (user-run)
```

Repo-specific facts
- Prerender keeps a page only if the body carries the TanStack Start SSR marker `$tsr-stream-barrier`. The port is not protection on its own: another project's `vite preview` can hold the same port, and a wildcard-IPv6 bind next to an IPv4 one lets both servers listen at once, so a liveness check passes while a foreign server answers. `scripts/deploy.sh` owns the port number and the bind address.
- Dev snapshot endpoint lives in `vite.config.ts` (`/__snapshot`) for deterministic render captures.
- Tension DPR is `GRID_SCALE` (pixel-stable grid), not raw `devicePixelRatio`.

---
> Source: [quantizor/art](https://github.com/quantizor/art) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
