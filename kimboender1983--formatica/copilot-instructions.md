## formatica

> Formatica is a schema-driven form builder shipped as three npm packages:

# Formatica — Project Instructions for Claude

## Project Overview

Formatica is a schema-driven form builder shipped as three npm packages:

- `@formatica/core` — Framework-agnostic engine (types, validation, schema parser, conditions). Zero deps.
- `@formatica/vue` — Vue 3 components + composables. Depends on core.
- `@formatica/react` — React components + hooks. Depends on core.

All packages live in a monorepo at the root of this repo using Yarn Berry workspaces.

## Monorepo Structure

```
formatica/
├── packages/
│   ├── core/          → @formatica/core (pure TS, no framework)
│   ├── vue/           → @formatica/vue (Vue 3 components, composables, plugin)
│   └── react/         → @formatica/react (React components, hooks, provider)
├── playground/        → Vue playground app (dev/demo site, deploys to Railway)
├── scripts/
│   └── publish.sh     → Version bump + build + test + publish to npm
├── Dockerfile         → Railway deployment (nginx serving playground)
├── biome.json         → Biome linter/formatter config
└── tsconfig.base.json → Shared TypeScript config
```

## Important: When Making ANY Change

Every change must be checked against ALL of these:

1. **Core package** (`packages/core/`) — Types, validation rules, schema parser, conditions, extractFields
2. **Vue package** (`packages/vue/`) — Components, composables, plugin, styles CSS
3. **React package** (`packages/react/`) — Components, hooks, provider
4. **Playground** (`playground/`) — Must still build and work after changes
5. **Tests** — Run all three test suites:
   - `yarn test --run` (Vue tests)
   - `yarn workspace @formatica/core test` (Core tests)
   - `yarn workspace @formatica/react test` (React tests)
6. **Documentation** — Update if the change affects the public API:
   - `playground/src/pages/DocsPage.vue` (platform-switched: Vue/React/Core)
   - `packages/core/README.md`
   - `packages/vue/README.md`
   - `packages/react/README.md`
   - `README.md` (root)
7. **Examples** (`playground/src/examples/`) — Update if schema format changes
8. **Package exports** — Update barrel exports (`src/index.ts`) in affected packages
9. **CSS file** (`packages/vue/src/styles/formatica.css`) — Update if component classes change

## Build Order

Core must build before Vue and React (they depend on it):

```bash
yarn build  # runs: core → vue → react
yarn build:playground
```

## Key Architecture Decisions

### Schema Format
The schema is a **unified tree** — fields and layout containers are the same `SchemaNode[]` array. No separate layout object.

```json
{
  "fields": [
    { "type": "text", "name": "email", "label": "Email" },
    { "type": "row", "children": [
      { "type": "text", "name": "city", "span": 8 },
      { "type": "text", "name": "zip", "span": 4 }
    ]}
  ]
}
```

### Theming
- Theme config generates CSS custom properties (`--fc-color-primary`, etc.) on the `<form>` element
- `packages/vue/src/styles/formatica.css` maps these vars to all components
- The CSS file MUST be imported by consumers: `import '@formatica/vue/style.css'`
- Dark mode uses `.dark` class strategy (Tailwind `darkMode: 'class'`)
- Component styles use inline `style` with `var(--fc-color-primary, #3b82f6)` for theme colors — NOT Tailwind color classes like `bg-blue-500`

### Plugin System
- **Vue**: `createFormatica(options)` — installed via `app.use()`
- **React**: `<FormaticaProvider config={...}>` — wraps the app
- **Core**: `configureFormatica(config)` — simple function
- Props on FormBuilder always override global config

### Field Registry
- Built-in types registered in FormBuilder on mount
- Custom types via: (1) plugin/provider `components` option, (2) FormBuilder `components` prop, (3) global `registerFieldType()`
- Resolution order: instance props → global registry

### Row Grid
- Rows use `grid-cols-12` CSS grid
- Fields use `span` property (1-12) for width
- Spans are applied via inline `style={{ gridColumn: 'span X' }}` — NOT dynamic Tailwind classes (they get purged)

## Code Style

- **Biome** for linting/formatting (not ESLint/Prettier)
- 4-space indent, double quotes, trailing commas
- `<script setup lang="ts">` for Vue components
- No `any` in public API types
- All Vue components use Tailwind CSS classes
- React components also use Tailwind CSS classes (matching Vue styling)

## Formatting & Linting

```bash
yarn format                    # format all files
yarn lint                      # check all files
yarn lint:fix                  # auto-fix
yarn biome format --write .    # format specific
```

## Testing

```bash
yarn test --run                          # Vue tests (114 tests)
yarn workspace @formatica/core test      # Core tests (97 tests)
yarn workspace @formatica/react test     # React tests (24 tests)
```

## Publishing

```bash
yarn publish          # patch bump + build + test + publish all 3 packages
yarn publish:minor    # minor bump
yarn publish:major    # major bump
```

## Playground

- Dev: `yarn dev` (Vite dev server)
- Build: `yarn build:playground`
- Deploy: Push to GitHub → Railway auto-deploys via Dockerfile
- The playground imports from `@formatica/vue` via workspace link
- Uses `@formatica/vue/style.css` for theming/dark mode

## Common Pitfalls

1. **Dynamic Tailwind classes get purged** — Never use `\`col-span-\${n}\`` — use inline `style` instead
2. **CSS layers** — Theme overrides in `formatica.css` must be OUTSIDE `@layer` to beat Tailwind utilities
3. **`workspace:*` in package.json** — Must be replaced with `^x.y.z` before npm publish. The publish script handles this, but `npm publish` (not `yarn npm publish`) is used because of 2FA.
4. **Vue SFC dark mode** — Don't rely on `dark:` Tailwind classes in components — they may not survive the build. Use the CSS file instead.
5. **prefers-color-scheme** — Don't use `@media (prefers-color-scheme: dark)` in formatica.css. Only use `.dark` class strategy.
6. **FormBuilder expose** — FormBuilder exposes `{ form }` via `defineExpose` so parent components can access form state via template ref.

## Playground Pages

- **Playground** (`/`) — Three-panel: schema editor, form preview, output panel (values/errors/state/events)
- **Builder** (`/builder`) — Visual drag-and-drop form builder
- **Examples** (`/examples`) — Gallery of 6 example forms with live preview
- **Docs** (`/docs/vue`, `/docs/react`, `/docs/core`) — Platform-switched API reference

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kimboender1983)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kimboender1983)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
