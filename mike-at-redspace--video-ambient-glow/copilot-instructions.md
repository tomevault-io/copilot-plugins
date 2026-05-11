## config

> Configuration files - TypeScript, ESLint, Rollup, Vitest


- TypeScript: strict mode, ES2022 target, ESNext modules, bundler resolution
- Rollup: builds both ESM (`index.js`) and CJS (`index.cjs`) from `src/index.ts`
- ESLint: TypeScript ESLint for `.ts` files, plain JS rules for `example/**/*.js`
- Vitest: `vitest.config.ts` (TypeScript), happy-dom environment, globals enabled, setup file at `tests/setup.ts`
- package.json: `type: "module"`, exports field for dual ESM/CJS, files array for publish
- Scripts follow pattern: `dev` (watch), `build` (production), `test`, `lint`, `format`
- Use `prepublishOnly` to run build + test before publishing
- Version scripts: `release:patch|minor|major` - auto-push tags
- ESLint ignores: `dist/`, `node_modules/`, `example/dist/`, `example/node_modules/`
- Prettier formats: TypeScript, JavaScript, JSON, Markdown, HTML, CSS, Handlebars
- @file tsconfig.json
- @file rollup.config.js
- @file eslint.config.js

---
> Source: [mike-at-redspace/video-ambient-glow](https://github.com/mike-at-redspace/video-ambient-glow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
