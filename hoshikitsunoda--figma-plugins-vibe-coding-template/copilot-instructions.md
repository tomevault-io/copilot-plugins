## figma-variables

> Variables (design tokens) enable dynamic theming, responsive design, and design system management.


# Figma Variables API

Variables (design tokens) enable dynamic theming, responsive design, and design system management.

## Core Concepts

- **Variable**: A named value that can change based on mode (e.g., light/dark theme)
- **Collection**: A group of related variables (e.g., "Colors", "Spacing")
- **Mode**: A variant of values within a collection (e.g., "Light", "Dark")

## Getting Variables

```typescript
// Get all local variable collections
const collections = await figma.variables.getLocalVariableCollectionsAsync()

// Get a specific variable by ID
const variable = await figma.variables.getVariableByIdAsync(variableId)

// Get variables from a collection
for (const collection of collections) {
  for (const variableId of collection.variableIds) {
    const variable = await figma.variables.getVariableByIdAsync(variableId)
    console.log(variable?.name, variable?.resolvedType)
  }
}
```

## Creating Variables

```typescript
// Create a new collection (sync)
const collection = figma.variables.createVariableCollection('My Tokens')

// Create variables of different types (pass VariableCollection object, not ID)
const colorVar = figma.variables.createVariable('primary', collection, 'COLOR')
const numberVar = figma.variables.createVariable(
  'spacing-md',
  collection,
  'FLOAT',
)
const stringVar = figma.variables.createVariable(
  'font-family',
  collection,
  'STRING',
)
const boolVar = figma.variables.createVariable(
  'is-enabled',
  collection,
  'BOOLEAN',
)

// Set values for the default mode
colorVar.setValueForMode(collection.defaultModeId, { r: 0.2, g: 0.4, b: 1 })
numberVar.setValueForMode(collection.defaultModeId, 16)
```

## Working with Modes

```typescript
// Add a new mode to a collection (sync)
const darkModeId = collection.addMode('Dark')

// Set values for different modes
colorVar.setValueForMode(collection.defaultModeId, { r: 1, g: 1, b: 1 }) // Light
colorVar.setValueForMode(darkModeId, { r: 0.1, g: 0.1, b: 0.1 }) // Dark

// Get value for a specific mode
const lightValue = colorVar.valuesByMode[collection.defaultModeId]
const darkValue = colorVar.valuesByMode[darkModeId]
```

## Applying Variables to Nodes

```typescript
// Bind a variable to a node's fill
const rect = figma.createRectangle()
const colorVar = await figma.variables.getVariableByIdAsync(variableId)

if (colorVar) {
  // Create a variable alias for the fill
  rect.fills = [
    {
      type: 'SOLID',
      color: { r: 0, g: 0, b: 0 }, // Fallback color
      boundVariables: {
        color: { type: 'VARIABLE_ALIAS', id: colorVar.id },
      },
    },
  ]
}
```

## Variable Types

| Type      | Description  | Value Format                                      |
| --------- | ------------ | ------------------------------------------------- |
| `COLOR`   | Color value  | `{ r: number, g: number, b: number, a?: number }` |
| `FLOAT`   | Number value | `number`                                          |
| `STRING`  | Text value   | `string`                                          |
| `BOOLEAN` | True/false   | `boolean`                                         |

## Variable Aliases (References)

Variables can reference other variables:

```typescript
// Create a semantic color that references a primitive
const semanticColor = figma.variables.createVariable(
  'bg-primary',
  collection,
  'COLOR',
)
semanticColor.setValueForMode(collection.defaultModeId, {
  type: 'VARIABLE_ALIAS',
  id: primitiveColorVar.id,
})
```

## Best Practices

1. **Use async getters** — Reading variables is async (`getVariableByIdAsync`, `getLocalVariableCollectionsAsync`)
2. **Creation is sync** — `createVariableCollection`, `createVariable`, `addMode` are synchronous
3. **Pass collection object** — `createVariable(name, collection, type)` takes the collection object, not ID
4. **Check for null** — Variables may not exist or may have been deleted
5. **Organize by collection** — Group related variables (colors, spacing, typography)
6. **Use semantic names** — Name variables by purpose, not value (`primary` not `blue-500`)

## MCP Tool

Use `figma_get_examples` with topic "variables" for more code examples.

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
