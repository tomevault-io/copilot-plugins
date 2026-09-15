## bg0

> BG0 is an open-source, browser-only background remover. The core flow is: open

# BG0 contributor instructions

BG0 is an open-source, browser-only background remover. The core flow is: open
BG0, choose an image, process it on-device, and download a transparent PNG.

## Product invariants

- Local mode is free, unlimited, anonymous, and the complete product.
- Image bytes, pixels, masks, filenames, dimensions, URLs, and thumbnails stay
  in the browser. Never send them to analytics, logs, or error reporting.
- Browser inference belongs behind `@bg0/browser`; UI code must not know ONNX
  internals.
- Authentication, billing, databases, hosted inference, and cloud APIs are not
  part of the current product.
- Keep BG0 focused on background removal. Do not add an editor, asset library,
  project system, or unrelated image features.
- Model and dependency licenses must permit commercial use and stay documented.

## Repository boundaries

- `apps/web`: TanStack Start site and anonymous local remover.
- `apps/docs`: static Blume documentation served under `/docs`.
- `packages/browser`: model caching, WebGPU/WASM selection, BiRefNet-lite
  inference, mask postprocessing, and PNG output.

Read `ARCHITECTURE.md` before changing image-data, analytics, model-hosting,
or deployment boundaries. Treat code and configuration as source of truth.

## Working rules

- Use Bun and root scripts. Keep `bun run dev`, `build`, `test`, `lint`, and
  `typecheck` working through Turborepo.
- Prefer small explicit modules and strict TypeScript.
- Use Tailwind tokens and repository shadcn primitives for common controls.
- Preserve keyboard access, visible focus, status announcements, contrast, and
  useful errors. Never surface raw runtime traces to users.
- Keep `.env.example` current. Never commit credentials, model caches, personal
  browser state, uploaded images, or downloads.
- Do not claim a physical device, deployed origin, WebGPU provider, or cache path
  works until that exact environment has been exercised.

## Verification

Run narrow checks while iterating, then root lint, typecheck, test, and build.

A rendered page or successful typecheck does not prove an interaction. Exercise
the affected user action, inspect client and server errors, and retain reviewable
evidence. Remover changes require a real browser flow with redistributable
fixtures: upload or drop, model preparation, processing, alpha-bearing PNG,
preview, download, reset, and a useful failure.

Track every process and fixture created by a test. Stop only owned processes and
delete only owned state. Agents share this checkout; inspect `git status`,
preserve unrelated work, and never reset or clean another worker's files.

---
> Source: [opencoredev/bg0](https://github.com/opencoredev/bg0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
