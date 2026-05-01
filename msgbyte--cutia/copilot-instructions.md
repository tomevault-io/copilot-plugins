## mobile-desktop-parity

> Ensure mobile and desktop editor stay in sync


# Mobile / Desktop Parity

The editor has two independent view layers sharing the same EditorCore and stores:

- **Desktop**: `components/editor/` (excluding `mobile/`)
- **Mobile**: `components/editor/mobile/`

## Rules

### When modifying editor features

Every change to editor behavior must work on **both** desktop and mobile:

1. If you add or change a feature in a desktop editor component, check if the corresponding mobile component exists and update it too.
2. If you add or change a feature in a mobile editor component, check if the corresponding desktop component needs the same change.
3. If you modify EditorCore, a Zustand store, a service, or a shared hook — verify both desktop and mobile views still work correctly.

### Mapping between desktop and mobile components

| Desktop | Mobile |
|---------|--------|
| `editor-layout.tsx` | `mobile/mobile-editor-layout.tsx` |
| `preview/` | `mobile/mobile-preview.tsx` |
| `timeline/` | `mobile/mobile-timeline/` |
| `properties/` | `mobile/mobile-drawer/mobile-properties-drawer.tsx` |
| `tools-panel/` (assets) | `mobile/mobile-drawer/mobile-assets-drawer.tsx` |
| Header bar | `mobile/mobile-header.tsx` |

### Shared layer (both platforms consume)

- `core/` (EditorCore and all managers)
- `stores/` (all Zustand stores)
- `services/` (renderer, storage, transcription, etc.)
- `hooks/actions/` (useEditorActions, useActionHandler)
- `types/`, `constants/`, `lib/`

Changes to the shared layer must not break either platform.

### Checklist for PRs touching editor code

- [ ] Desktop component updated?
- [ ] Corresponding mobile component updated?
- [ ] Shared layer changes tested for both?
- [ ] Touch interactions preserved on mobile?
- [ ] Keyboard/mouse interactions preserved on desktop?

---
> Source: [msgbyte/cutia](https://github.com/msgbyte/cutia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
