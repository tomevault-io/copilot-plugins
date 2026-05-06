## components

> Pre-built components using Radix UI + Tailwind:

# Component Guidelines

## UI Component Library

### shadcn/ui Components (src/components/ui/)
Pre-built components using Radix UI + Tailwind:
- `button.tsx` - Primary actions
- `input.tsx`, `textarea.tsx` - Form inputs
- `select.tsx` - Dropdowns
- `sheet.tsx` - Slide-out panels (use for forms)
- `dialog.tsx` - Modal dialogs
- `alert-dialog.tsx` - Confirmations
- `tabs.tsx` - Tab navigation
- `table.tsx` - Data tables
- `badge.tsx` - Status indicators
- `card.tsx` - Content containers
- `dropdown-menu.tsx` - Context menus
- `popover.tsx` - Floating content
- `tooltip.tsx` - Hover hints
- `checkbox.tsx` - Boolean inputs
- `progress.tsx` - Progress bars

### Adding New shadcn Components
```bash
npx shadcn@latest add [component-name]
```

## Icon Usage

### Primary: Tabler Icons
```tsx
import { IconPlus, IconTrash, IconEdit } from '@tabler/icons-react';

<IconPlus className="icon-sm" />  // 16px
<IconEdit className="icon-md" />  // 20px
<IconTrash className="icon-lg" /> // 24px
```

### Common Icons
- Add: `IconPlus`
- Edit: `IconPencil`, `IconEdit`
- Delete: `IconTrash`
- Save: `IconDeviceFloppy`
- Cancel: `IconX`
- Search: `IconSearch`
- Filter: `IconFilter`
- Sort: `IconArrowsSort`
- Expand: `IconChevronDown`
- Collapse: `IconChevronUp`
- Menu: `IconDotsVertical`
- Camera: `IconCamera`
- Photo: `IconPhoto`
- Upload: `IconUpload`
- Download: `IconDownload`
- Check: `IconCheck`
- Warning: `IconAlertTriangle`
- Info: `IconInfoCircle`

## Project-Specific Components

### Project Tabs (src/components/project/tabs/)
- `deal-summary-tab.tsx` - Financials, ROI, MAO
- `budget-detail-tab.tsx` - Three-column budget table
- `vendors-tab.tsx` - Vendor directory
- `draws-tab.tsx` - Payment tracking
- `cost-reference-tab.tsx` - Minneapolis pricing

### Form Sheets (Slide-out panels)
- `vendor-form-sheet.tsx` - Add/edit vendors
- `budget-item-form-sheet.tsx` - Add budget items
- `draw-form-sheet.tsx` - Add/edit draws
- `photo-upload-sheet.tsx` - Upload photos

### Detail Sheets (Read-only views)
- `vendor-detail-sheet.tsx` - Full vendor info

## Dashboard Components (src/components/dashboard/)

- `portfolio-health.tsx` - Hero metrics cards
- `kanban-pipeline.tsx` - Drag-drop project board
- `project-card.tsx` - Pipeline project cards
- `stat-card.tsx` - Metric display card
- `animated-number.tsx` - Counting animation

## Component Patterns

### Sheet Form Pattern
```tsx
interface FormSheetProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  item?: ItemType;  // If provided, edit mode
  onSuccess?: () => void;
}

export function ItemFormSheet({ open, onOpenChange, item, onSuccess }: FormSheetProps) {
  const isEditing = !!item;
  const mutation = useItemMutation();
  
  const handleSubmit = async (data: ItemInput) => {
    await mutation.mutateAsync(data);
    onOpenChange(false);
    onSuccess?.();
  };
  
  return (
    <Sheet open={open} onOpenChange={onOpenChange}>
      <SheetContent>
        <SheetHeader>
          <SheetTitle>{isEditing ? 'Edit' : 'Add'} Item</SheetTitle>
        </SheetHeader>
        {/* Form content */}
      </SheetContent>
    </Sheet>
  );
}
```

### Confirmation Dialog Pattern
```tsx
<AlertDialog open={deleteOpen} onOpenChange={setDeleteOpen}>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Delete Item?</AlertDialogTitle>
      <AlertDialogDescription>
        This action cannot be undone.
      </AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Cancel</AlertDialogCancel>
      <AlertDialogAction onClick={handleDelete} className="bg-destructive">
        Delete
      </AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

### Data Table Pattern
```tsx
// Use TanStack Table for complex tables
import { useReactTable, getCoreRowModel, flexRender } from '@tanstack/react-table';

const table = useReactTable({
  data,
  columns,
  getCoreRowModel: getCoreRowModel(),
});
```

## CSS Utility Classes (from globals.css)

### Status Badges
```tsx
<span className="status-badge status-active">Active</span>
<span className="status-badge status-pending">Pending</span>
<span className="status-badge status-completed">Complete</span>
```

### Stat Cards
```tsx
<div className="stat-card">
  <span className="stat-label">Total Budget</span>
  <span className="stat-value">$125,000</span>
</div>
```

### Tables
```tsx
<thead className="table-header">
<tr className="table-row-hover">
```

### Icons
```tsx
<IconPlus className="icon-sm" />  // 16px
<IconPlus className="icon-md" />  // 20px
<IconPlus className="icon-lg" />  // 24px
```

### Animations
```tsx
<div className="fade-in">...</div>
<div className="scale-in">...</div>
<div className="slide-in-bottom">...</div>
```

## Responsive Design

### Breakpoints (Tailwind v4)
- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px
- `2xl`: 1536px

### Common Patterns
```tsx
// Mobile-first grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">

// Hide on mobile
<div className="hidden md:block">

// Stack on mobile, row on desktop
<div className="flex flex-col md:flex-row">
```

---
> Source: [adamj-ops/rehab-budget-pro](https://github.com/adamj-ops/rehab-budget-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
