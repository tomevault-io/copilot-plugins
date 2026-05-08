## figma-export

> Export nodes as images or retrieve image data from fills.


# Figma Export API

Export nodes as images or retrieve image data from fills.

## Exporting Nodes

### Basic Export

```typescript
const node = figma.currentPage.selection[0]
if (!node) return

// Export as PNG
const pngBytes = await node.exportAsync({
  format: 'PNG',
  constraint: { type: 'SCALE', value: 2 } // 2x resolution
})

// Export as SVG
const svgBytes = await node.exportAsync({
  format: 'SVG'
})

// Export as JPG
const jpgBytes = await node.exportAsync({
  format: 'JPG',
  constraint: { type: 'SCALE', value: 1 }
})

// Export as PDF
const pdfBytes = await node.exportAsync({
  format: 'PDF'
})
```

### Export Constraints

```typescript
// Scale constraint (multiplier)
{ type: 'SCALE', value: 2 } // 2x size

// Width constraint (fixed width, height scales proportionally)
{ type: 'WIDTH', value: 200 } // 200px wide

// Height constraint (fixed height, width scales proportionally)
{ type: 'HEIGHT', value: 100 } // 100px tall
```

### Export Settings

```typescript
// Full export settings
const bytes = await node.exportAsync({
  format: 'PNG',
  constraint: { type: 'SCALE', value: 1 },
  contentsOnly: true, // Exclude frame itself, only children
  useAbsoluteBounds: false, // Use node bounds, not absolute
})
```

## Sending to UI for Download

```typescript
// In plugin (main.ts)
const bytes = await node.exportAsync({ format: 'PNG' })
figma.ui.postMessage({
  type: 'download-image',
  data: Array.from(bytes), // Convert Uint8Array to regular array
  filename: `${node.name}.png`
})

// In UI (App.tsx)
window.addEventListener('message', (event) => {
  const msg = event.data.pluginMessage
  if (msg.type === 'download-image') {
    const blob = new Blob([new Uint8Array(msg.data)], { type: 'image/png' })
    const url = URL.createObjectURL(blob)
    
    const link = document.createElement('a')
    link.href = url
    link.download = msg.filename
    link.click()
    
    URL.revokeObjectURL(url)
  }
})
```

## Working with Image Fills

### Get Image Data from Fill

```typescript
if ('fills' in node && Array.isArray(node.fills)) {
  for (const fill of node.fills) {
    if (fill.type === 'IMAGE' && fill.imageHash) {
      const image = figma.getImageByHash(fill.imageHash)
      if (image) {
        const bytes = await image.getBytesAsync()
        const size = await image.getSizeAsync()
        console.log(`Image size: ${size.width}x${size.height}`)
      }
    }
  }
}
```

### Set Image Fill

```typescript
// In UI: Convert image to bytes and send to plugin
const input = document.createElement('input')
input.type = 'file'
input.accept = 'image/*'
input.onchange = async () => {
  const file = input.files?.[0]
  if (file) {
    const buffer = await file.arrayBuffer()
    parent.postMessage({
      pluginMessage: {
        type: 'set-image',
        data: Array.from(new Uint8Array(buffer))
      }
    }, '*')
  }
}
input.click()

// In plugin: Create image and apply as fill
figma.ui.onmessage = async (msg) => {
  if (msg.type === 'set-image') {
    const bytes = new Uint8Array(msg.data)
    const image = figma.createImage(bytes)
    
    const rect = figma.createRectangle()
    rect.resize(200, 200)
    rect.fills = [{
      type: 'IMAGE',
      imageHash: image.hash,
      scaleMode: 'FILL'
    }]
  }
}
```

## Image Scale Modes

```typescript
const imageFill: ImagePaint = {
  type: 'IMAGE',
  imageHash: image.hash,
  scaleMode: 'FILL', // Options: 'FILL', 'FIT', 'CROP', 'TILE'
}

// For CROP mode, you can set image transform
const croppedFill: ImagePaint = {
  type: 'IMAGE',
  imageHash: image.hash,
  scaleMode: 'CROP',
  imageTransform: [[1, 0, 0], [0, 1, 0]] // 2D transform matrix
}
```

## Batch Export

```typescript
async function exportSelection(format: 'PNG' | 'SVG' | 'JPG') {
  const selection = figma.currentPage.selection
  const exports: { name: string; data: number[] }[] = []
  
  for (const node of selection) {
    const bytes = await node.exportAsync({ format })
    exports.push({
      name: `${node.name}.${format.toLowerCase()}`,
      data: Array.from(bytes)
    })
  }
  
  figma.ui.postMessage({ type: 'batch-export', exports })
}
```

## Best Practices

1. **Convert Uint8Array for postMessage** — Use `Array.from(bytes)`
2. **Handle large exports** — Consider chunking or progress indicators
3. **Check node exportability** — Some nodes may not export correctly
4. **Use appropriate format** — PNG for raster, SVG for vectors

## MCP Tool

Use `figma_get_examples` with topic "export" for more code examples.

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
