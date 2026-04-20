## arete-builder

> Modal system patterns using Zustand store


# Modal System Patterns

## Modal Registration

All modals must be registered in [components/dialogs/Modals.tsx](mdc:components/dialogs/Modals.tsx):

```typescript
// Register new modal in the registry
export const modalRegistry = {
  createProject: CreateEditProjectDialog,
  editProject: CreateEditProjectDialog,
  createPage: CreateEditPageDialog,
  // Add new modals here
} as const;
```

## Opening Modals

Use [lib/store/modalStore.ts](mdc:lib/store/modalStore.ts) to open modals:

```typescript
// In any component
const modalStore = useModalStore();

// Open modal with props
modalStore.openModal("createProject", {
  project: null,
  onSave: handleCreate,
  isLoading: mutation.isPending,
});

// Open edit modal
modalStore.openModal("editProject", {
  project: selectedProject,
  onSave: handleUpdate,
  isLoading: updateMutation.isPending,
});
```

## Modal Component Pattern

Modal components should follow this pattern:

```typescript
// components/dialogs/ExampleDialog.tsx
interface ExampleDialogProps {
  isOpen: boolean;
  onClose: () => void;
  // Additional props passed from openModal
  item?: Item;
  onSave: (data: ItemData) => void;
  isLoading?: boolean;
}

export function ExampleDialog({ isOpen, onClose, item, onSave, isLoading }: ExampleDialogProps) {
  return (
    <Dialog open={isOpen} onOpenChange={onClose}>
      <DialogContent>
        {/* Dialog content */}
      </DialogContent>
    </Dialog>
  );
}
```

## Modal Naming Convention

- Use `CreateEdit*Dialog` pattern for create/edit modals
- Modal keys should be descriptive: `createProject`, `editPage`, `deleteConfirm`
- Props are passed through Zustand store, not direct component props

## State Management

- Modal state managed by Zustand store in [lib/store/modalStore.ts](mdc:lib/store/modalStore.ts)
- Modal props are passed as second argument to `openModal()`
- Use `closeModal()` to close active modal
- Check `modalStore.activeModal` for current modal state

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JohnSyoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
