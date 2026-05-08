## figma-performance

> Handling large documents efficiently in the Plugin API.


# Figma Performance Patterns

Handling large documents efficiently in the Plugin API.

## Skip Invisible Instance Children

Speed up traversal by skipping hidden nested instances:

```typescript
// ✓ Set before traversing large documents
figma.skipInvisibleInstanceChildren = true
```

## Use Built-in Find Methods

Figma's find methods are optimized internally:

```typescript
// ✓ Preferred — uses internal optimization
const frames = page.findAll((n) => n.type === 'FRAME')
const firstText = page.findOne((n) => n.type === 'TEXT')

// ❌ Avoid manual recursion when possible
function findAllManual(node) { ... }
```

## Destructure Repeated Access

Avoid multiple property lookups on the same object:

```typescript
// ✓ Destructure once
const { x, y, width, height } = node.absoluteBoundingBox

// ❌ Repeated access
const x = node.absoluteBoundingBox.x
const y = node.absoluteBoundingBox.y
```

## Batch with Yielding

For very large operations, yield to prevent UI freeze:

```typescript
async function processNodes(nodes: readonly SceneNode[]) {
  for (const node of nodes) {
    processNode(node)
    // Yield to main thread periodically
    await new Promise((r) => setTimeout(r, 0))
  }
}
```

## Load Pages On Demand

Pages are loaded dynamically — only access what you need:

```typescript
// ✓ Work with current page when possible
const nodes = figma.currentPage.findAll(...)

// Loading other pages is async
await figma.loadAllPagesAsync()
for (const page of figma.root.children) {
  // Now all pages are accessible
}
```

## Avoid Large Payloads

When sending data to UI, serialize only what's needed:

```typescript
// ✓ Send minimal data
figma.ui.postMessage({
  type: 'nodes',
  data: nodes.map((n) => ({ id: n.id, name: n.name })),
})

// ❌ Don't serialize entire node trees unnecessarily
```

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
