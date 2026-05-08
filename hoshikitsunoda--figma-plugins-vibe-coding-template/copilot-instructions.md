## figma-serialization

> Converting Plugin API data to JSON for postMessage or export.


# Figma Serialization Patterns

Converting Plugin API data to JSON for postMessage or export.

## Clone Readonly Arrays

Paint and effect arrays are readonly — clone before modifying or sending:

```typescript
// ✓ Clone with JSON round-trip
const fills = JSON.parse(JSON.stringify(node.fills))

// ✓ Or use spread for shallow clone
const effects = [...node.effects]
```

## Handle Special Types

### Colors

Figma colors use 0-1 range, not 0-255:

```typescript
interface Color {
  r: number // 0-1
  g: number // 0-1
  b: number // 0-1
}

// Convert to hex if needed
const toHex = (c: Color) =>
  '#' +
  [c.r, c.g, c.b]
    .map((v) => Math.round(v * 255).toString(16).padStart(2, '0'))
    .join('')
```

### Image Paints

Image fills contain `imageHash`, not actual bytes:

```typescript
if (fill.type === 'IMAGE') {
  // imageHash is a string reference
  const hash = fill.imageHash

  // To get actual bytes, use:
  const image = figma.getImageByHash(hash)
  const bytes = await image.getBytesAsync()
}
```

### Font Names

```typescript
if (node.type === 'TEXT') {
  // FontName is { family: string, style: string }
  const font = node.fontName
  // May be mixed — check for figma.mixed symbol
  if (font !== figma.mixed) {
    console.log(font.family, font.style)
  }
}
```

## Skip Non-Serializable Properties

These will break `JSON.stringify()`:

```typescript
// ❌ Don't include in serialized output
node.parent // Circular reference
node.removed // Runtime state
node.reactions // Contains functions

// ✓ Extract only needed data
function serializeNode(node: SceneNode) {
  return {
    id: node.id,
    name: node.name,
    type: node.type,
    // ... other safe properties
  }
}
```

## Mixed Values

Some properties can be `figma.mixed` when values vary (e.g., mixed fonts in text):

```typescript
if (node.fontSize === figma.mixed) {
  // Handle mixed case
} else {
  // Use the value directly
}
```

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
