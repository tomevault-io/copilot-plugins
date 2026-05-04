## tools

> Editor tools structure in apps/editor


# Tools

Tools are React components that capture user input (pointer, keyboard) and translate it into `useScene` mutations. They live exclusively in `apps/editor/components/tools/`.

## Lifecycle

`ToolManager` reads `useEditor` (phase + mode + tool) and mounts the active tool component. When the tool changes, the old component unmounts, cleaning up any transient state.

See @apps/editor/components/tools/tool-manager.tsx.

## Tool Categories by Phase

**Site**
- `site-boundary-editor` — draw/edit property boundary polygon

**Structure**
- `wall-tool` — draw walls segment by segment
- `slab-tool` + `slab-boundary-editor` + `slab-hole-editor`
- `ceiling-tool` + `ceiling-boundary-editor` + `ceiling-hole-editor`
- `roof-tool`
- `door-tool` + `door-move-tool`
- `window-tool` + `window-move-tool`
- `item-tool` + `item-move-tool`
- `zone-tool` + `zone-boundary-editor`

**Furnish**
- `item-tool` — place furniture

**Shared utilities**
- `polygon-editor` — reusable boundary/hole editing logic
- `cursor-sphere` — 3D cursor visualisation

## Pattern

```tsx
// apps/editor/components/tools/my-tool/index.tsx
import { useScene } from '@pascal-app/core'
import { useEditor } from '../../store/use-editor'

export function MyTool() {
  const createNode = useScene(s => s.createNode)
  const setTool = useEditor(s => s.setTool)

  // Pointer handlers mutate the scene store directly.
  // No local geometry — use a renderer for any preview mesh.

  return (
    <mesh onPointerDown={handleDown} onPointerMove={handleMove}>
      {/* ghost / preview geometry only */}
    </mesh>
  )
}
```

## Rules

- **Tools only mutate `useScene`** — they do not call Three.js APIs directly.
- **No business logic in tools** — delegate geometry/constraint rules to core systems.
- **Preview geometry is local** — transient meshes shown while a tool is active live in the tool component, not in the scene store.
- **Clean up on unmount** — remove any pending/incomplete nodes when the tool unmounts.
- **Tools must not import from `@pascal-app/viewer`** — use the scene store and core hooks only.
- Each tool should handle a single, well-scoped interaction. Split complex tools (e.g. "draw + move") into separate components selected by `useEditor`.

## Adding a New Tool

1. Create `apps/editor/components/tools/<name>/index.tsx`.
2. Register the tool in `ToolManager` under the correct phase and mode.
3. Add the tool identifier to the `useEditor` tool union type.
4. If the tool requires new node types, add schema + renderer + system first.

---
> Source: [zuleikdiasb-web/pascal-editor-3d](https://github.com/zuleikdiasb-web/pascal-editor-3d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
