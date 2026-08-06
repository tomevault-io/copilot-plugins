## slexkit

> SlexKit — Streaming Live EXpressions Kit. Zero-build, Markdown-friendly reactive UI runtime for AI output. This file tells AI agents how to work in this repository.

# AGENTS.md

SlexKit — Streaming Live EXpressions Kit. Zero-build, Markdown-friendly reactive UI runtime for AI output. This file tells AI agents how to work in this repository.

## Tech stack

- **Runtime**: TypeScript (ES2022, strict mode, ESNext modules)
- **Components**: Svelte 5 (runes syntax, `$props()`, `$state()`)
- **Styling**: UnoCSS (wind4 preset) with shadcn/ui design tokens; per-component CSS files in `src/styles/components/`
- **Build**: Bun.build with custom Svelte plugin; TSC for declarations only
- **Test**: `bun test` with jsdom browser environment
- **Package manager**: Bun (workspaces in `packages/`)
- **Site**: Svelte 5 + Bun HTTP server + CodeMirror + KaTeX + Flowbite-Svelte

## Key commands

```sh
bun run dev           # Start site dev server
bun run build         # Build all (core + streamdown + mcp)
bun run build:core    # Build core runtime only
bun test              # Run all tests
bun run test:watch    # Run tests in watch mode
bun run lint          # ESLint (src/ only)
bun run format        # Prettier (src/ only)
bun run smoke:release # Release smoke test
```

## Directory map

```
src/
  engine/           # Core runtime: renderer, reactive system, expression eval, store, merge, diagnostics, component-state, markdown-runtime, secure-runtime, sandbox-runner
  components/       # Component registration + Svelte sources (layout/ content/ display/ input/ disclosure/ feedback/ tooling/)
  toolhost/         # AI tool-call rendering (confirm-action, choose-options, fill-form)
  icons/            # Phosphor icon system + custom registration API
  styles/           # CSS: layout, theme, content, display, input, disclosure, feedback, animation, tooling + per-component files

packages/
  runtime/          # @slexkit/runtime — thin wrapper re-exporting slexkit/runtime
  components-svelte/# @slexkit/components-svelte — thin wrapper re-exporting slexkit/components-svelte
  theme-shadcn/     # @slexkit/theme-shadcn — CSS only
  streamdown/       # @slexkit/streamdown — React/Streamdown custom renderer
  mcp/              # @slexkit/mcp — read-only docs/examples/validate MCP server

site/               # Documentation website (Svelte-based SPA)
  content/          # Markdown docs: guides/ + components/
  components/       # Site-specific Svelte components
  pages/            # Page definitions as .slex.js DSL files
  scripts/          # Site build and static export

tests/              # Bun test suites (runtime/, components/, secure/, site/, toolhost/, markdown/)
examples/           # Integration examples (basic-resistor, component-gallery, custom-component, markdown-fence, secure-fetch, toolhost)
scripts/            # Build (build-core.ts), CLI (cli.mjs), release-smoke (release-smoke.mjs)
```

## Code conventions

### Component registration pattern
1. Import `.svelte` component
2. Wrap with `createSvelteRenderer(type, Component)` in `adapter.ts`
3. `register(type, renderer, { state })` stores in global registry
4. At render time, `renderTree()` looks up by `type` from `getRenderer()`

### Reactive state flow
1. `SlexExpression` contains `g` (reactive state object) and `layout` (component tree)
2. `createReactiveState(g)` wraps it in deep Proxy for dependency tracking
3. Components receive `$prop` expressions evaluated via `evalRead()` (`new Function()`)
4. Write expressions (`onchange`) execute via `execWrite()` inside `batch()`

### Theme architecture
1. CSS custom properties from shadcn/ui tokens (`--background`, `--foreground`, `--primary`, etc.)
2. Three theme modes: `uno` (built-in), `host-shadcn` (inherits host tokens), `flowbite`
3. `uno.config.ts` generates CSS variable declarations for light/dark
4. `.slexkit-theme-host-shadcn` generates primary color scale via `color-mix()`

### Naming
- Component keys: `type:identifier` (e.g. `"card:greeting"`, `"text:message"`)
- Files: kebab-case (`component-state.ts`, `secure-runtime.ts`)
- Public API: camelCase (`mountSecureArtifact`, `createSecureRuntime`)
- Svelte components: PascalCase (`Accordion.svelte`, `Card.svelte`)

### Key exports (src/index.ts)
- **Engine**: `mount`, `ingest`, `boot`, `disposeNamespace`, `register`, `getRenderer`, `mountSecureArtifact`, `parseSlexSource`, `createSecureRuntime`, `createSlexKitMarkdownRuntimeHost`
- **Components**: `registerAll`, `registerSvelteComponent`, `registerSubset`
- **ToolHost**: `renderToolCall`, `registerToolTemplate`
- **Icons**: `registerIcon`, `registerIcons`, `clearRegisteredIcons`, `getIcon`, `getRegisteredIcon`, `loadIcon`, `normalizeIconName`, `resolveIconWeight`, `resolveIconifyIcon`, `iconifySvgUrl`
- **Lifecycle**: `attachComponentDisposer`
- **Diagnostics**: `diagnoseSlexKitSource`, `formatSlexKitDiagnostic`, `SlexKitSyntaxError`, `SlexKitRuntimeError`
- **Sandbox**: `startSlexKitSandboxRunner`, `setSlexKitRuntimeUrl`, `getSlexKitRuntimeUrl`

### Adding a new component
1. Create `.svelte` file in `src/components/svelte/<category>/`
2. Create corresponding CSS in `src/styles/components/` (import from `src/styles/input.css` or similar)
3. Register in `src/components/index.ts` with appropriate `ComponentStateMode`
4. Add component doc page in `site/content/components/<name>.md` (Chinese)
5. Add entry to `publicComponentSlugs` in `site/data/component-docs.js`
6. Add tests in `tests/components/`

### Writing tests
- Tests use Bun test runner with `--conditions browser --isolate --preload ./tests/setup.ts`
- `tests/setup.ts` provides a jsdom DOM environment with mocked browser APIs
- Use `mountDsl()` from `tests/helpers.ts` to mount DSL expressions in tests
- Test files are organized by domain: `tests/runtime/`, `tests/components/`, `tests/secure/`, etc.

## Documentation conventions

- All `docs/*.md` are in English — these are the authoritative technical docs
- `site/content/` is in Chinese — user-facing documentation site
- Package READMEs exist for each package in `packages/<name>/README.md`
- Component docs in `site/content/components/` use YAML frontmatter with `title`, `category`, `status`, `order`, `summary`
- Guide docs in `site/content/guides/` use YAML frontmatter with similar fields
- `site/data/component-docs.js` is the metadata engine — sync frontmatter data with it when adding guides/components

---
> Source: [slexkit/slexkit](https://github.com/slexkit/slexkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
