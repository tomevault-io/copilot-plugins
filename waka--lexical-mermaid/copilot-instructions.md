## lexical-mermaid

> This document contains important information for developing and maintaining the Lexical Mermaid plugin.

# Lexical Mermaid Plugin - Development Guide

This document contains important information for developing and maintaining the Lexical Mermaid plugin.

## Project Overview

A Mermaid diagram embedding plugin for Lexical Editor. Allows you to add interactive, click-to-edit Mermaid diagrams to your rich text editor.

### Tech Stack
- **TypeScript 5.3** - Type safety
- **React 18** - UI implementation
- **Lexical 0.39** - Meta's rich text editor framework
- **Mermaid 11** - Diagram rendering library
- **tsup** - Fast build tool (ESM/CJS support)
- **Storybook 10** - Component development and testing environment
- **ESLint 9** - Code quality and linting (flat config)

## Architecture

### Directory Structure

```
src/
├── nodes/
│   └── MermaidNode.tsx           # DecoratorNode implementation (core)
├── components/
│   ├── MermaidComponent.tsx      # Display component with export buttons
│   └── MermaidEditor.tsx         # Edit component
├── plugins/
│   ├── MermaidPlugin.tsx         # Insert from button
│   └── MermaidMarkdownPlugin.tsx # Insert via ```mermaid syntax
├── stories/
│   └── MermaidEditor.stories.tsx # Storybook stories
├── utils/
│   ├── mermaidRenderer.ts        # Mermaid initialization and rendering
│   └── exportUtils.ts            # SVG/PNG export functionality
└── types/
    └── index.ts                  # Type definitions
```

### Core Components

#### 1. MermaidNode (DecoratorNode)
- Extends Lexical's DecoratorNode
- Holds Mermaid code and handles serialization/deserialization
- Renders MermaidComponent via `decorate()` method

**Important**: `updateDOM()` always returns `false` (to re-render with React)

#### 2. MermaidComponent
- Display and edit mode toggle for Mermaid diagrams
- Click to enter edit mode
- Renders Mermaid in `useEffect`
- Shows SVG/PNG export buttons on hover
- Export buttons appear in top-right corner when hovering over diagram

**Important**: Generate unique `elementId` each time (using `Date.now()`)
- Mermaid has issues with rendering multiple times with the same ID

**Export functionality**:
- Hover to reveal export buttons (SVG and PNG)
- Buttons use `stopPropagation()` to prevent entering edit mode
- Export only available when SVG is successfully rendered (no error state)

#### 3. MermaidEditor
- Edit Mermaid code in textarea
- Save, Cancel, Delete buttons
- Keyboard shortcuts (Ctrl+Enter: save, Esc: cancel)

#### 4. MermaidPlugin
- Registers `INSERT_MERMAID_COMMAND`
- Automatically creates next paragraph and focuses after node insertion

#### 5. MermaidMarkdownPlugin
- Monitors Enter key
- Detects `` ```mermaid `` pattern and auto-converts
- Replaces current paragraph directly (prevents unnecessary line breaks)

#### 6. Export Utilities (exportUtils.ts)
- `exportSVG()` - Export diagram as SVG file
- `exportPNG()` - Convert SVG to PNG using Canvas API and export
- Automatic timestamp-based filename generation
- White background for PNG (avoid transparency)

**PNG Conversion Process**:
1. Parse SVG string to extract dimensions (width/height or viewBox)
2. Create Image element and load SVG as data URL
3. Draw image on Canvas with white background
4. Convert Canvas to PNG Blob
5. Trigger download with generated filename

## Important Design Decisions

### 1. Using peerDependencies

**Problem**: Including `lexical` in dependencies causes duplicate instances and errors

**Solution**: Move all dependencies to `peerDependencies`
```json
{
  "dependencies": {},
  "peerDependencies": {
    "@lexical/react": ">=0.17.0",
    "lexical": ">=0.17.0",
    "mermaid": ">=11.0.0",
    "react": ">=18.0.0",
    "react-dom": ">=18.0.0"
  }
}
```

### 2. Node Update Method

**Problem**: `node.setMermaidCode(newCode)` alone doesn't trigger re-render

**Solution**: Replace entire node with a new one
```typescript
const newNode = $createMermaidNode(newCode)
node.replace(newNode)
```

### 3. Mermaid Rendering

**Problem**: Rendering multiple times with same `elementId` fails

**Solution**: Generate unique ID each time
```typescript
const elementId = `mermaid-${nodeKey}-${Date.now()}`
```

### 4. Security Level

Recommend `securityLevel: 'loose'` for development, `'strict'` for production
```typescript
mermaid.initialize({
  securityLevel: config?.securityLevel || 'loose',
  // ...
})
```

## Development Notes

### Development with Storybook

Use Storybook to test the plugin.

**How to start**:
```bash
# Start Storybook dev server (http://localhost:6006)
npm run storybook

# Build Storybook statically
npm run build-storybook
```

**Story placement**:
- Place stories in `src/stories/` directory
- Use `.stories.tsx` or `.stories.ts` extension
- Not included in build (not in `tsup` `entry`)

**Storybook configuration**:
- `.storybook/main.ts` - Story search patterns and Vite config
- `.storybook/preview.ts` - Global settings

**Important**: Storybook is for development only. Excluded from npm publish via `files: ["dist"]`

### Code Quality with ESLint

ESLint 9 with flat config format is used for code quality checks.

**How to run**:
```bash
# Check all TypeScript files in src/
npm run lint

# Run type checking
npm run type-check
```

**Configuration file**: `eslint.config.js`

**Key ESLint rules**:
- TypeScript recommended rules enabled
- `no-undef` disabled (TypeScript handles this)
- Unused variables error (prefix with `_` to allow)
- React JSX Transform support (no need to import React in components)
- Storybook files have relaxed rules

**Best practices**:
- Run `npm run lint` before committing
- Fix all errors (don't commit with lint errors)
- Use `npm run type-check` to verify types without building
- Prefix intentionally unused variables with `_` (e.g., `_unusedVar`)

**Common patterns to avoid**:
```typescript
// ❌ Bad - unused imports
import { TextNode, SerializedNode } from 'lexical'

// ✅ Good - only import what you use
import { $createParagraphNode } from 'lexical'
```

### Building

```bash
# Build plugin (ESM/CJS/type definitions)
npm run build

# Development with watch mode
npm run dev
```

## Troubleshooting

### Error: "does not subclass LexicalNode"

**Cause**: Duplicate Lexical instances

**Solution**:
1. Remove `node_modules` and reinstall
2. Verify peerDependencies are correctly configured
3. Restart dev server/Storybook

### Storybook Error: "Failed to fetch dynamically imported module"

**Cause**: Mermaid dynamic import failed

**Solution**:
Add Mermaid to optimization targets in `.storybook/main.ts` `viteFinal`:
```typescript
async viteFinal(config) {
  return {
    ...config,
    optimizeDeps: {
      ...config.optimizeDeps,
      include: [...(config.optimizeDeps?.include || []), 'mermaid'],
    },
  }
}
```

### Chart Disappears

**Cause**:
- On save: Incorrect node update method
- On cancel: Duplicate `elementId`

**Solution**:
- On save: Use `node.replace(newNode)`
- On cancel: Generate unique ID with `Date.now()`

## Usage

### Basic Usage

```typescript
import { LexicalComposer } from '@lexical/react/LexicalComposer'
import { MermaidNode, MermaidPlugin, MermaidMarkdownPlugin, initMermaid } from 'lexical-mermaid'

const editorConfig = {
  nodes: [MermaidNode],
  // ...
}

function Editor() {
  useEffect(() => {
    initMermaid() // Initialize beforehand
  }, [])

  return (
    <LexicalComposer initialConfig={editorConfig}>
      <MermaidPlugin />
      <MermaidMarkdownPlugin />
      {/* ... */}
    </LexicalComposer>
  )
}
```

### Insert from Button

```typescript
import { INSERT_MERMAID_COMMAND, getDefaultMermaidCode } from 'lexical-mermaid'

function InsertButton() {
  const [editor] = useLexicalComposerContext()

  const onClick = () => {
    editor.dispatchCommand(INSERT_MERMAID_COMMAND, getDefaultMermaidCode())
  }

  return <button onClick={onClick}>Insert Mermaid</button>
}
```

### Markdown Syntax

Type the following in the editor:
```
```mermaid
[Enter]
```

Automatically inserts a Mermaid node.

## Extensibility

### Adding New Templates

Define `templates` object in Storybook stories or consumer code:

```typescript
const templates = {
  newTemplate: `graph LR
    A --> B`,
}

// Usage example
editor.dispatchCommand(INSERT_MERMAID_COMMAND, templates.newTemplate)
```

### Custom Theme

```typescript
initMermaid({
  theme: 'dark',
  // Other settings
})
```

### Adding Validation

Extend `validateMermaidCode()` in `src/utils/mermaidRenderer.ts`

## Project Structure

### File Structure

```
lexical-mermaid/
├── .storybook/          # Storybook config (dev only)
├── dist/                # Build output (gitignored)
├── src/
│   ├── components/      # React components
│   ├── nodes/           # Lexical nodes
│   ├── plugins/         # Lexical plugins
│   ├── stories/         # Storybook stories (not in build)
│   ├── types/           # TypeScript type definitions
│   ├── utils/           # Utilities
│   └── index.ts         # Entry point (exports)
├── .gitignore           # Git exclusion settings
├── eslint.config.js     # ESLint configuration (flat config)
├── package.json
├── tsconfig.json
└── tsup.config.ts       # Build configuration
```

### Files Included in npm Publish

With `package.json`'s `files: ["dist"]`, only the following are published:
- `dist/index.js` - ESM build
- `dist/index.cjs` - CJS build
- `dist/index.d.ts` - Type definitions (ESM)
- `dist/index.d.cts` - Type definitions (CJS)
- `package.json`, `README.md`, `LICENSE` (automatically included)

### .gitignore Key Points

```gitignore
node_modules/
dist/
storybook-static/
*.log
```

Development environment files are excluded from Git.

## Performance Optimization

- Mermaid is a heavy library, so initialize only when needed
- Call `initMermaid()` only once (managed with global flag)
- Tree-shaking with `tsup` at build time removes unused code
- Exclude peerDependencies from bundle with `external` setting

## Future Improvements

- [ ] Real-time preview (preview while editing)
- [ ] Accessibility improvements
- [ ] Mobile support improvements
- [ ] Clipboard copy functionality (copy SVG/PNG to clipboard)

## References

- [Lexical Documentation](https://lexical.dev/)
- [Mermaid Documentation](https://mermaid.js.org/)
- [DecoratorNode Guide](https://lexical.dev/docs/concepts/nodes#decorator-nodes)

## License

MIT License

---
> Source: [waka/lexical-mermaid](https://github.com/waka/lexical-mermaid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
