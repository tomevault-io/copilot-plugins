## figma-styles

> Styles are reusable design properties that can be applied to multiple nodes.


# Figma Styles API

Styles are reusable design properties that can be applied to multiple nodes.

## Style Types

| Style Type | Applies To | Properties |
|------------|------------|------------|
| Paint Style | Fills, strokes | Colors, gradients, images |
| Text Style | Text nodes | Font, size, line height, etc. |
| Effect Style | Any node | Shadows, blurs |
| Grid Style | Frames | Layout grids |

## Getting Local Styles

```typescript
// Get all local styles (async)
const paintStyles = await figma.getLocalPaintStylesAsync()
const textStyles = await figma.getLocalTextStylesAsync()
const effectStyles = await figma.getLocalEffectStylesAsync()
const gridStyles = await figma.getLocalGridStylesAsync()

// Log style names
paintStyles.forEach(style => console.log(style.name))
```

## Creating Styles

### Paint Style

```typescript
const style = figma.createPaintStyle()
style.name = "Brand/Primary"
style.paints = [{
  type: 'SOLID',
  color: { r: 0.2, g: 0.4, b: 1 }
}]
```

### Text Style

```typescript
const style = figma.createTextStyle()
style.name = "Heading/H1"
style.fontName = { family: "Inter", style: "Bold" }
style.fontSize = 32
style.lineHeight = { value: 40, unit: "PIXELS" }
style.letterSpacing = { value: -0.5, unit: "PIXELS" }
```

### Effect Style

```typescript
const style = figma.createEffectStyle()
style.name = "Elevation/Medium"
style.effects = [{
  type: 'DROP_SHADOW',
  color: { r: 0, g: 0, b: 0, a: 0.15 },
  offset: { x: 0, y: 4 },
  radius: 8,
  spread: 0,
  visible: true,
  blendMode: 'NORMAL'
}]
```

## Applying Styles to Nodes

```typescript
// Apply paint style to fills
const rect = figma.createRectangle()
rect.fillStyleId = paintStyle.id

// Apply paint style to strokes
rect.strokeStyleId = strokeStyle.id

// Apply text style
if (textNode.type === "TEXT") {
  textNode.textStyleId = textStyle.id
}

// Apply effect style
node.effectStyleId = effectStyle.id

// Apply grid style to frame
if (frame.type === "FRAME") {
  frame.gridStyleId = gridStyle.id
}
```

## Getting Style from Node

```typescript
// Check if node uses a style
if (node.fillStyleId && typeof node.fillStyleId === 'string') {
  const style = await figma.getStyleByIdAsync(node.fillStyleId)
  console.log("Fill style:", style?.name)
}

// Handle mixed styles (multiple different styles applied)
if (node.fillStyleId === figma.mixed) {
  console.log("Node has mixed fill styles")
}
```

## Importing Styles from Libraries

```typescript
// Import a style by key
const importedStyle = await figma.importStyleByKeyAsync(styleKey)
node.fillStyleId = importedStyle.id
```

## Updating Styles

```typescript
// Modify an existing style
const style = await figma.getStyleByIdAsync(styleId)
if (style?.type === "PAINT") {
  style.paints = [{
    type: 'SOLID',
    color: { r: 1, g: 0, b: 0 } // Change to red
  }]
}
```

## Style Properties

### PaintStyle
- `paints`: Array of Paint objects
- `name`: Style name
- `description`: Optional description

### TextStyle
- `fontName`: `{ family: string, style: string }`
- `fontSize`: number
- `lineHeight`: `{ value: number, unit: "PIXELS" | "PERCENT" | "AUTO" }`
- `letterSpacing`: `{ value: number, unit: "PIXELS" | "PERCENT" }`
- `textCase`: `"ORIGINAL" | "UPPER" | "LOWER" | "TITLE"`
- `textDecoration`: `"NONE" | "UNDERLINE" | "STRIKETHROUGH"`

### EffectStyle
- `effects`: Array of Effect objects (shadows, blurs)

## Best Practices

1. **Use async getters** — `getLocalPaintStylesAsync()` instead of sync versions
2. **Check for mixed** — Style IDs can be `figma.mixed` on text ranges
3. **Organize with `/`** — Use slashes in names for hierarchy: `"Colors/Primary/500"`
4. **Handle remote styles** — Library styles have `remote: true`

## MCP Tool

Use `figma_get_examples` with topic "styles" for more code examples.

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
