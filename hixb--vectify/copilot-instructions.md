## vectify

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vectify is a CLI tool that generates framework-specific icon components from SVG files. It supports 10+ frameworks (React, Vue, Svelte, Solid.js, Preact, Angular, Lit, Qwik, Astro, Vanilla JS) with full TypeScript support and automatic SVG optimization.

**Key characteristics:**
- Package manager: `pnpm@9.15.9`
- Node requirement: `>=18.0.0`
- Built with `tsup` (outputs both CJS and ESM)
- Template-based code generation using Handlebars

## Development Commands

### Build
```bash
pnpm build          # Build for production (CJS + ESM + types)
pnpm dev            # Build in watch mode
pnpm clean          # Remove dist directory
```

### Testing
```bash
pnpm test           # Run tests once
pnpm test:watch     # Run tests in watch mode
```

### Linting
```bash
pnpm lint           # Check for lint errors
pnpm lint:fix       # Fix lint errors automatically
```

### Testing Generated Icons (Manual)
```bash
# After making changes, test icon generation:
pnpm build
cd /tmp && mkdir test-vectify && cd test-vectify
npm init -y
npm install /path/to/vectify/dist

# Create test SVG files
mkdir icons
echo '<svg viewBox="0 0 24 24"><path d="M12 2L2 7v10l10 5 10-5V7z"/></svg>' > icons/test.svg

# Initialize and generate
npx vectify init    # Select framework, configure paths
npx vectify generate
```

## Architecture Overview

### Core Data Flow

The generation pipeline follows this flow:

```
SVG File → beforeParse Hook → SVGO Optimization → SVG Parser (Cheerio)
→ IconNode Array → Framework Strategy → Template Rendering (Handlebars)
→ afterGenerate Hook → File Writer → Formatter → onComplete Hook
```

### Key Architectural Patterns

#### 1. **IconNode Format** (Core Data Structure)

All SVG elements are converted to `IconNode` tuples for framework-agnostic representation:

```typescript
type IconNode = [
  SVGElementType, // 'path' | 'circle' | 'rect' | 'g' etc.
  Record<string, string | number>, // Attributes (camelCase)
  IconNode[]? // Optional children
]
```

Example:
```typescript
// SVG: <circle cx="12" cy="12" r="10" stroke-width="2"/>
// IconNode: ['circle', { cx: 12, cy: 12, r: 10, strokeWidth: 2 }]
```

This format is:
- Serializable to component code
- Framework-agnostic
- Compact for embedding in generated files

#### 2. **Strategy Pattern** (Framework Abstraction)

`src/generators/framework-strategy.ts` defines the `FrameworkStrategy` interface that all framework generators implement:

```typescript
interface FrameworkStrategy {
  name: Framework
  getComponentExtension: (typescript: boolean) => string
  getIndexExtension: (typescript: boolean) => string
  generateComponent: (name, iconNode, typescript, keepColors) => string
  generateBaseComponent: (typescript) => { code: string, fileName: string }
}
```

**Why this matters:**
- Adding a new framework requires implementing this interface
- Each framework has different base components:
  - React: `createIcon.tsx` (factory function)
  - Vue: `Icon.vue` (base component)
  - Svelte: `Icon.svelte` (base component)
  - Angular: `icon.component.ts` (base component class)

All strategies are registered in `FrameworkRegistry` singleton at `src/generators/framework-strategy.ts:339-393`.

#### 3. **Template Engine** (Code Generation)

Handlebars templates (`src/generators/templates/**/*.hbs`) separate logic from presentation.

**Important build detail:** Templates are NOT bundled by tsup. They're copied to `dist/templates/` via `scripts/copy-templates.js` (triggered by tsup's `onSuccess` hook in `tsup.config.ts:11`).

**Template path resolution** (`src/generators/templates/template-engine.ts:23-33`):
- CJS: Uses `__dirname` → `dist/templates/`
- ESM: Uses `import.meta.url` → `dist/templates/`

**Template structure:**
```
templates/
├── react/
│   ├── component.tsx.hbs      # Individual icon component
│   └── createIcon.tsx.hbs     # Base helper function
├── vue/
│   ├── component.ts.vue.hbs
│   └── icon.ts.vue.hbs        # Base Icon.vue component
├── svelte/
├── solid/
├── preact/
├── qwik/
├── angular/
├── lit/
├── astro/
└── vanilla/
```

Each template receives:
- `typescript`: boolean
- `componentName`: string (PascalCase)
- `formattedNodes`: string (IconNode array as code string)
- `keepColors`: boolean (preserve SVG colors vs. currentColor)

#### 4. **Config Loading with `jiti`**

`src/config/loader.ts` uses `jiti` to dynamically load TypeScript config files at runtime (no transpilation step needed). This allows users to write `vectify.config.ts` directly.

**Search order:**
1. `--config` CLI flag path
2. `vectify.config.ts`
3. `vectify.config.js`

**Path resolution:** All `input`/`output` paths are resolved relative to `configDir` (defaults to config file location).

### Critical Files by Function

| Function | File Path | Key Responsibilities |
|----------|-----------|---------------------|
| CLI Entry | `src/cli.ts` | Commander.js setup, command registration |
| Type Definitions | `src/types.ts` | `IconForgeConfig`, `IconNode`, `Framework` types |
| Main Orchestrator | `src/generators/index.ts` | Pipeline coordination, file writing, hooks execution |
| SVG Parsing | `src/parsers/svg-parser.ts` | Cheerio-based SVG → IconNode conversion |
| SVG Optimization | `src/parsers/optimizer.ts` | SVGO integration |
| Framework Registry | `src/generators/framework-strategy.ts` | Strategy pattern registry |
| Template Engine | `src/generators/templates/template-engine.ts` | Handlebars rendering, template path resolution |
| Code Formatting | `src/utils/formatter.ts` | Auto-detection and execution (Biome > Prettier > ESLint) |
| File Operations | `src/utils/helpers.ts` | File I/O, naming (PascalCase conversion), directory ops |

### Export Styles by Framework

**Default exports** (use `export default`):
- Vue
- Svelte
- Preact

**Named exports** (use `export const`):
- React
- Solid
- Qwik
- Angular
- Astro
- Vanilla JS
- Lit

**Why this matters:** The index file generator (`src/generators/index.ts:generateIndexFile`) automatically uses the correct export syntax based on framework.

### SVG to Component Generation Details

**Attribute name transformation:**
- Dash-case → camelCase: `stroke-width` → `strokeWidth`
- Numeric strings converted to numbers: `"12"` → `12`
- Namespace attributes removed: `xmlns`, `xmlns:xlink`

**Color handling:**
- `keepColors: false` (default): Replaces `fill`/`stroke` with `currentColor`
- `keepColors: true`: Preserves original SVG colors

**Component naming:**
1. Strip `.svg` extension
2. Convert to PascalCase: `arrow-right.svg` → `ArrowRight`
3. Apply `prefix` and `suffix` if configured
4. Custom `transform` function overrides all above

## Adding a New Framework

To add support for a new framework:

1. **Create generator** in `src/generators/<framework>.ts`:
   ```typescript
   import { IconNode } from '../types'
   import { formatIconNode } from '../parsers/svg-parser'
   import { renderTemplate, get<Framework>TemplatePath } from './templates/template-engine'

   export function generate<Framework>Component(
     componentName: string,
     iconNode: IconNode[],
     typescript: boolean,
     keepColors: boolean
   ): string {
     const formattedNodes = formatIconNode(iconNode, 0, keepColors)
     const templatePath = get<Framework>TemplatePath(typescript, 'component')
     return renderTemplate(templatePath, {
       typescript,
       componentName,
       formattedNodes,
       keepColors
     })
   }

   export function generate<Framework>BaseComponent(typescript: boolean): string {
     // Generate base component/helper
   }
   ```

2. **Create templates** in `src/generators/templates/<framework>/`:
   - `component.ts.hbs` (or `.tsx.hbs`, `.vue.hbs` etc.)
   - Base component template (e.g., `createIcon.ts.hbs`, `icon.vue.hbs`)

3. **Add template path helper** in `src/generators/templates/template-engine.ts`:
   ```typescript
   export function get<Framework>TemplatePath(
     typescript: boolean,
     type: 'component' | 'baseComponent'
   ): string {
     const ext = typescript ? 'ts' : 'js'
     return `<framework>/${type}.${ext}.hbs`
   }
   ```

4. **Create strategy class** in `src/generators/framework-strategy.ts`:
   ```typescript
   class <Framework>Strategy implements FrameworkStrategy {
     name: Framework = '<framework>'

     getComponentExtension = (typescript: boolean): string => {
       return typescript ? 'ts' : 'js' // or 'tsx', 'vue', etc.
     }

     getIndexExtension = (typescript: boolean): string => {
       return typescript ? 'ts' : 'js'
     }

     generateComponent = (
       componentName: string,
       iconNode: IconNode[],
       typescript: boolean,
       keepColors = false
     ): string => {
       return generate<Framework>Component(componentName, iconNode, typescript, keepColors)
     }

     generateBaseComponent = (typescript: boolean) => {
       return {
         code: generate<Framework>BaseComponent(typescript),
         fileName: 'BaseComponent.<ext>'
       }
     }
   }
   ```

5. **Register strategy** in `FrameworkRegistry` constructor:
   ```typescript
   constructor() {
     // ... existing registrations
     this.register(new <Framework>Strategy())
   }
   ```

6. **Update types** in `src/types.ts`:
   ```typescript
   export type Framework = 'react' | 'vue' | 'svelte' | /* ... */ | '<framework>'
   ```

7. **Test** by generating icons for the new framework and verifying output.

## Testing Philosophy

Since this is a code generator, testing focuses on:
- **Output verification:** Does generated code match expected templates?
- **IconNode parsing:** Are SVGs correctly converted to IconNode format?
- **Framework strategy:** Does each strategy generate valid code?
- **Edge cases:** Empty SVGs, malformed SVGs, special characters in names

When adding tests, place them in `__tests__/` directories adjacent to the code being tested.

## Common Gotchas

### 1. Template Changes Require Rebuild
Templates are copied during build (`tsup onSuccess` hook). If you modify `.hbs` files:
```bash
pnpm build  # Templates won't update without rebuilding
```

### 2. Path Resolution in Monorepos
Users often misconfigure paths in monorepos. The `configDir` option exists to resolve this:
```typescript
// apps/web/vectify.config.ts
export default defineConfig({
  configDir: '../..', // Relative to monorepo root
  input: '../../icons', // Now resolved correctly
  output: './src/icons'
})
```

### 3. React "use client" Directive
React components include `'use client'` at the top (`src/generators/templates/react/component.tsx.hbs`) for Next.js App Router compatibility. This is intentional and framework-specific.

### 4. SVGO Default Config
The default SVGO config removes `viewBox` and dimensions (`src/parsers/optimizer.ts`). This is intentional to allow runtime size control via props. Users can override via `svgoConfig` option.

### 5. Formatter Auto-Detection
`src/utils/formatter.ts` auto-detects formatters by checking for config files:
- `biome.json` / `biome.jsonc` → Biome
- `.prettierrc*` / `prettier.config.*` → Prettier
- `eslint.config.*` / `.eslintrc*` → ESLint

Priority: Biome > Prettier > ESLint

## Configuration Philosophy

Vectify prioritizes sensible defaults:
- TypeScript by default (`typescript: true`)
- Optimization enabled (`optimize: true`)
- Single-color icons (`keepColors: false` → uses `currentColor`)
- Index file generation (`generateOptions.index: true`)

Users only configure what differs from defaults.

---
> Source: [hixb/vectify](https://github.com/hixb/vectify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
