## figma-components

> Components enable reusable design elements with variants and overrides.


# Figma Components API

Components enable reusable design elements with variants and overrides.

## Core Concepts

- **ComponentNode**: A reusable design element (the "main" component)
- **ComponentSetNode**: A container for component variants
- **InstanceNode**: A copy of a component that inherits from the main

## Creating Components

```typescript
// Create a frame first
const frame = figma.createFrame()
frame.resize(100, 40)
frame.name = "Button"
frame.fills = [{ type: 'SOLID', color: { r: 0.2, g: 0.4, b: 1 } }]

// Convert to component
const component = figma.createComponentFromNode(frame)
// Note: 'frame' is now invalid - use 'component' instead
```

## Creating Instances

```typescript
// From a component reference
const instance = component.createInstance()
instance.x = component.x + 150

// From component key (for library components)
const importedComponent = await figma.importComponentByKeyAsync(componentKey)
const instance2 = importedComponent.createInstance()
```

## Component Variants

```typescript
// Create variant components
const defaultButton = figma.createComponent()
defaultButton.name = "State=Default"

const hoverButton = figma.createComponent()
hoverButton.name = "State=Hover"

// Combine into a component set
const componentSet = figma.combineAsVariants([defaultButton, hoverButton], figma.currentPage)
componentSet.name = "Button"
```

## Working with Variants

```typescript
// Get variant properties from a component set
if (node.type === "COMPONENT_SET") {
  // Parse variant properties from child names
  for (const variant of node.children) {
    console.log(variant.name) // e.g., "Size=Large, State=Hover"
  }
}

// Swap instance to different variant
if (instance.type === "INSTANCE") {
  const mainComponent = await instance.getMainComponentAsync()
  const componentSet = mainComponent?.parent
  
  if (componentSet?.type === "COMPONENT_SET") {
    // Find the variant you want
    const targetVariant = componentSet.children.find(
      child => child.name === "Size=Large, State=Default"
    )
    if (targetVariant?.type === "COMPONENT") {
      instance.swapComponent(targetVariant)
    }
  }
}
```

## Instance Overrides

```typescript
// Override text in an instance
if (instance.type === "INSTANCE") {
  // Find text node inside instance
  const textNode = instance.findOne(n => n.type === "TEXT")
  if (textNode?.type === "TEXT") {
    await figma.loadFontAsync(textNode.fontName as FontName)
    textNode.characters = "New Label"
  }
}

// Reset overrides
instance.resetOverrides()
```

## Detaching Instances

```typescript
// Detach an instance to make it editable
if (instance.type === "INSTANCE") {
  const detached = instance.detachInstance()
  // 'detached' is now a FrameNode
}
```

## Getting Component Information

```typescript
// Get main component from instance
if (instance.type === "INSTANCE") {
  const main = await instance.getMainComponentAsync()
  console.log(main?.name, main?.key)
}

// Check if node is from external library
if (instance.type === "INSTANCE") {
  const main = await instance.getMainComponentAsync()
  if (main?.remote) {
    console.log("This is from an external library")
  }
}
```

## Component Properties

```typescript
// Get component properties (for instances)
if (instance.type === "INSTANCE") {
  const props = instance.componentProperties
  for (const [name, prop] of Object.entries(props)) {
    console.log(name, prop.type, prop.value)
  }
}

// Set component property value
instance.setProperties({
  "Show Icon": true,
  "Label": "Click Me"
})
```

## Best Practices

1. **Use async for main component** — `getMainComponentAsync()` is required
2. **Check remote status** — Library components behave differently
3. **Handle missing components** — Main component may be null if deleted
4. **Load fonts before text changes** — Even in instances

## MCP Tool

Use `figma_get_examples` with topic "components" for more code examples.

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
