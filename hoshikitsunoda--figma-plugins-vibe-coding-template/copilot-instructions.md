## figma-node-properties

> Safe patterns for accessing node properties in the Plugin API.


# Figma Node Properties

Safe patterns for accessing node properties in the Plugin API.

## Type Narrowing

Check `node.type` before accessing type-specific properties:

```typescript
// ✓ Type narrowing by node.type
if (node.type === "TEXT") {
  console.log(node.characters);
  console.log(node.fontSize);
}

if (node.type === "FRAME") {
  console.log(node.layoutMode);
  console.log(node.children);
}
```

## Property Existence Check

Use `in` operator for properties shared across multiple node types:

```typescript
// ✓ Check property exists before access
if ('fills' in node) {
  console.log(node.fills)
}

if ('children' in node) {
  node.children.forEach((child) => ...)
}

if ('absoluteBoundingBox' in node) {
  const { x, y, width, height } = node.absoluteBoundingBox
}
```

## Common Mixins

Properties are grouped by "mixins" — capabilities shared across node types:

| Mixin         | Properties                            | Node Types            |
| ------------- | ------------------------------------- | --------------------- |
| GeometryMixin | `fills`, `strokes`, `strokeWeight`    | Shapes, frames, text  |
| LayoutMixin   | `layoutMode`, `primaryAxisAlignItems` | Frames, components    |
| BlendMixin    | `opacity`, `blendMode`, `effects`     | Most visible nodes    |
| ChildrenMixin | `children`                            | Frames, groups, pages |

## Global Properties

These exist on all `SceneNode` types:

- `id` — Unique identifier
- `name` — Layer name
- `type` — Node type string
- `parent` — Parent node (or null)
- `visible` — Visibility state

---
> Source: [hoshikitsunoda/figma-plugins-vibe-coding-template](https://github.com/hoshikitsunoda/figma-plugins-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
