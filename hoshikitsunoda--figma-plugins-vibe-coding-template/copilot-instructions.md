## figma-dev-mode

> Dev Mode provides developer-focused views and Code Connect links design to code.


# Figma Dev Mode & Code Connect

Dev Mode provides developer-focused views and Code Connect links design to code.

## Dev Mode Detection

```typescript
// Check if plugin is running in Dev Mode
if (figma.mode === 'dev') {
  console.log('Running in Dev Mode')
  // Show developer-focused UI
} else {
  console.log('Running in Design Mode')
}
```

## Code Connect Overview

Code Connect links Figma components to their code implementations, showing relevant code snippets in Dev Mode.

> **Note:** Code Connect is typically configured via a separate `figma.config.json` file and CLI, not through the Plugin API. See https://www.figma.com/developers/code-connect

## Dev Mode Properties

### Dev Resources

```typescript
// Get dev resources attached to a node (async)
const resources = await node.getDevResourcesAsync()
for (const resource of resources) {
  console.log(resource.name, resource.url)
}

// Add a dev resource (link to code, docs, etc.)
await node.addDevResourceAsync(
  'https://github.com/org/repo/blob/main/src/Button.tsx',
  'Component Source',
)

// Edit an existing dev resource
await node.editDevResourceAsync(
  'https://github.com/org/repo/blob/main/src/Button.tsx',
  { name: 'Updated Name', url: 'https://new-url.com' },
)

// Delete a dev resource
await node.deleteDevResourceAsync(
  'https://github.com/org/repo/blob/main/src/Button.tsx',
)
```

### Annotations

```typescript
// Dev Mode annotations are read-only in plugins
// They're managed through Figma's UI
```

## Measurements & Specs

```typescript
// Get node measurements
if ('absoluteBoundingBox' in node) {
  const bounds = node.absoluteBoundingBox
  console.log(`Position: ${bounds.x}, ${bounds.y}`)
  console.log(`Size: ${bounds.width} x ${bounds.height}`)
}

// Get computed styles for dev handoff
function getNodeSpecs(node: SceneNode) {
  const specs: Record<string, unknown> = {
    name: node.name,
    type: node.type,
  }

  if ('absoluteBoundingBox' in node) {
    specs.bounds = node.absoluteBoundingBox
  }

  if ('fills' in node && Array.isArray(node.fills)) {
    specs.fills = node.fills.filter((f) => f.visible !== false)
  }

  if ('effects' in node) {
    specs.effects = node.effects.filter((e) => e.visible !== false)
  }

  if ('cornerRadius' in node) {
    specs.cornerRadius = node.cornerRadius
  }

  if (node.type === 'TEXT') {
    specs.fontSize = node.fontSize
    specs.fontName = node.fontName
    specs.lineHeight = node.lineHeight
    specs.letterSpacing = node.letterSpacing
  }

  return specs
}
```

## CSS Generation

```typescript
// Generate CSS from node properties
function generateCSS(node: SceneNode): string {
  const css: string[] = []

  if ('absoluteBoundingBox' in node) {
    css.push(`width: ${node.absoluteBoundingBox.width}px;`)
    css.push(`height: ${node.absoluteBoundingBox.height}px;`)
  }

  if ('cornerRadius' in node && typeof node.cornerRadius === 'number') {
    css.push(`border-radius: ${node.cornerRadius}px;`)
  }

  if ('fills' in node && Array.isArray(node.fills)) {
    const solidFill = node.fills.find(
      (f) => f.type === 'SOLID' && f.visible !== false,
    )
    if (solidFill && solidFill.type === 'SOLID') {
      const { r, g, b } = solidFill.color
      const a = solidFill.opacity ?? 1
      css.push(
        `background-color: rgba(${Math.round(r * 255)}, ${Math.round(g * 255)}, ${Math.round(b * 255)}, ${a});`,
      )
    }
  }

  if ('effects' in node) {
    const shadow = node.effects.find(
      (e) => e.type === 'DROP_SHADOW' && e.visible !== false,
    )
    if (shadow && shadow.type === 'DROP_SHADOW') {
      const { r, g, b, a } = shadow.color
      css.push(
        `box-shadow: ${shadow.offset.x}px ${shadow.offset.y}px ${shadow.radius}px rgba(${Math.round(r * 255)}, ${Math.round(g * 255)}, ${Math.round(b * 255)}, ${a});`,
      )
    }
  }

  return css.join('\n')
}
```

## Generating Code Snippets

```typescript
// Generate React component code from a frame
function generateReactComponent(node: FrameNode): string {
  const name = node.name.replace(/[^a-zA-Z0-9]/g, '')

  return `
import React from 'react';

export function ${name}() {
  return (
    <div style={{
      width: ${node.width},
      height: ${node.height},
      display: 'flex',
      flexDirection: '${node.layoutMode === 'VERTICAL' ? 'column' : 'row'}',
      gap: ${node.itemSpacing || 0},
      padding: '${node.paddingTop}px ${node.paddingRight}px ${node.paddingBottom}px ${node.paddingLeft}px',
    }}>
      {/* Children */}
    </div>
  );
}
`.trim()
}
```

## Best Practices

1. **Check mode** — Adapt UI based on `figma.mode`
2. **Provide dev resources** — Link to relevant code/docs
3. **Generate accurate specs** — Include all relevant properties
4. **Handle design tokens** — Map variables to CSS custom properties

## MCP Tool

Use `figma_search_api` with query "dev mode" for more documentation.

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
