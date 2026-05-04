## viewer-isolation

> Viewer must be editor-agnostic — controlled from outside via props and children


# Viewer Isolation

`@pascal-app/viewer` is a standalone 3D canvas library. It must never know about editor-specific features, UI state, or tools. This keeps it usable in the read-only `/viewer/[id]` route and in any future embedding context.

## The Rule

> The viewer is controlled from outside. It exposes control points (props, callbacks, children). It never reaches into `apps/editor`.

## Forbidden in `packages/viewer`

```ts
// ❌ Never import from the editor app
import { useEditor } from '@/store/use-editor'
import { ToolManager } from '@/components/tools/tool-manager'

// ❌ Never reference editor-specific concepts
if (isEditorMode) { … }
```

## Correct Pattern — Pass Control from Outside

The editor mounts the viewer and passes what it needs:

```tsx
// apps/editor/components/editor-canvas.tsx  ✅
import { Viewer } from '@pascal-app/viewer'
import { ToolManager } from '../tools/tool-manager'
import { useEditor } from '../../store/use-editor'

export function EditorCanvas() {
  const { selection } = useViewer()

  return (
    <Viewer
      theme="light"
      onSelect={(id) => useViewer.getState().setSelection(id)}
      onExport={handleExport}
    >
      {/* Editor injects tools as children — viewer renders them inside the canvas */}
      <ToolManager />
    </Viewer>
  )
}
```

The viewer accepts `children` and renders them inside the R3F canvas. This is the extension point for tools, overlays, and editor-specific systems.

## Viewer's Own State (`useViewer`)

The viewer store contains **only presentation state**:

- `selection` — which nodes are highlighted
- `cameraMode` — perspective / orthographic
- `levelMode` — stacked / exploded / solo / manual
- `wallMode` — up / cutaway / down
- `theme` — light / dark
- Display toggles: `showScans`, `showGuides`, `showGrid`

If a piece of state is only meaningful inside the editor (e.g. active tool, phase, edit mode) — it belongs in `useEditor`, not `useViewer`.

## Nested Viewer for Editor-Specific Features

When an editor feature needs to live "inside" the canvas but must not pollute the viewer package, inject it as a child:

```tsx
// ✅ Editor-specific overlay injected as child
<Viewer>
  <SelectionBoxOverlay />   {/* editor only */}
  <SnapIndicator />         {/* editor only */}
  <ToolManager />           {/* editor only */}
</Viewer>
```

This pattern lets the viewer stay ignorant of these components while they still have access to the R3F context.

## Checklist Before Adding Code to `packages/viewer`

- [ ] Does this feature make sense in the read-only viewer route?
- [ ] Does it reference `useEditor`, tool state, or phase/mode?
- [ ] Could it be passed in as a prop or child instead?

If any answer is "editor-specific", keep it in `apps/editor` and inject it via children or props.

---
> Source: [zuleikdiasb-web/pascal-editor-3d](https://github.com/zuleikdiasb-web/pascal-editor-3d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
