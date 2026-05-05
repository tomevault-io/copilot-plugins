## dashboard-ui-design-principles

> Design conventions for RBIP dashboard panels with flat hierarchy, progressive disclosure, and modern UX patterns.

# Dashboard UI Design Principles

Design conventions for RBIP dashboard panels with flat hierarchy, progressive disclosure, and modern UX patterns.

## Core Philosophy

- **Flat hierarchy**: Maximum 2 nesting levels (page → section/card)
- **Progressive disclosure**: Show summary first, reveal details on interaction
- **Clear confirmation**: Actions require explicit user confirmation, especially destructive ones
- **Visual clarity**: Use spacing, dividers, and typography instead of nested borders

---

## Layout & Hierarchy

### Flat Structure

```tsx
// ❌ Nested Card (bad)
<Card>
  <Card>
    <CardHeader>Title</CardHeader>
    <CardContent>Content</CardContent>
  </Card>
</Card>

// ✅ Flat layout with dividers (good)
<div className="space-y-6">
  <section>
    <h3 className="text-lg font-semibold">Section Title</h3>
    <hr className="border-border my-4" />
    <div>Content here</div>
  </section>
</div>
```

### Section Headers

- Use uppercase tracking for sub-section headers
- Font: `text-xs font-semibold text-muted-foreground uppercase tracking-wide`
- Margin: `mb-3` for headers, `mb-2` for helper text

```tsx
<h4 className="text-xs font-semibold text-muted-foreground uppercase tracking-wide mb-3">
  SECTION HEADER
</h4>
```

### Dividers

- Use `<hr className="border-border" />` for horizontal separation
- Use `divide-y` or `divide-x` for lists
- Avoid nested border containers

---

## Progressive Disclosure

### Collapsible Rank Groups

Use chevron icons with collapsible sections:

```tsx
// State
const [collapsedRankGroups, setCollapsedRankGroups] = useState<Set<'therapists' | 'pca'>>(new Set(['pca']))

// Toggle handler
const toggleRankGroup = (group: 'therapists' | 'pca') => {
  setCollapsedRankGroups(prev => {
    const next = new Set(prev)
    if (next.has(group)) {
      next.delete(group)
    } else {
      next.add(group)
    }
    return next
  })
}

// Render
<div className="flex items-center gap-2">
  <button onClick={() => toggleRankGroup('therapists')}>
    {collapsedRankGroups.has('therapists') ? (
      <ChevronRight className="h-4 w-4" />
    ) : (
      <ChevronDown className="h-4 w-4" />
    )}
  </button>
  <span className="text-sm font-medium">THERAPISTS</span>
</div>
```

### Per-Item Expand/Collapse

For individual staff items within a group:

```tsx
const [expandedStaffIds, setExpandedStaffIds] = useState<Set<string>>(new Set())

// Toggle
const toggleStaffExpand = (staffId: string) => {
  setExpandedStaffIds(prev => {
    const next = new Set(prev)
    next.has(staffId) ? next.delete(staffId) : next.add(staffId)
    return next
  })
}

// Render with chevron
<div className="flex items-center gap-2">
  <button onClick={() => toggleStaffExpand(id)}>
    {expandedStaffIds.has(id) ? (
      <ChevronDown className="h-4 w-4" />
    ) : (
      <ChevronRight className="h-4 w-4" />
    )}
  </button>
  <span>{staffName}</span>
</div>
```

---

## Checkbox Selection with Confirmation

### Multi-Select Staff List

```tsx
const [staffIdsToAdd, setStaffIdsToAdd] = useState<Set<string>>(new Set())

// Toggle selection
const toggleStaffSelection = (staffId: string) => {
  setStaffIdsToAdd(prev => {
    const next = new Set(prev)
    next.has(staffId) ? next.delete(staffId) : next.add(staffId)
    return next
  })
}

// Render checklist (filter inactive, separate regular/buffer)
<div className="max-h-40 overflow-y-auto bg-muted/30 rounded p-2 pr-1 scrollbar-visible">
  {regularStaff.map(s => (
    <label key={s.id} className="flex items-center space-x-2 py-1 cursor-pointer hover:bg-muted/50 rounded px-1">
      <input
        type="checkbox"
        checked={staffIdsToAdd.has(s.id)}
        onChange={() => toggleStaffSelection(s.id)}
      />
      <span>{s.name} ({s.rank})</span>
    </label>
  ))}
  {/* Divider before buffer staff */}
  {bufferStaff.length > 0 && regularStaff.length > 0 && (
    <hr className="border-border my-2" />
  )}
  {bufferStaff.map(s => (
    <label key={s.id} className="flex items-center space-x-2 py-1 cursor-pointer hover:bg-muted/50 rounded px-1">
      <input type="checkbox" checked={staffIdsToAdd.has(s.id)} onChange={() => toggleStaffSelection(s.id)} />
      <span>{s.name} (Floating, Buffer)</span>
    </label>
  ))}
</div>
```

### Confirmation Action Buttons

```tsx
{staffIdsToAdd.size > 0 && (
  <div className="flex items-center gap-2 mt-3">
    <Button
      size="sm"
      className="h-8 px-3 text-xs font-medium bg-[#0f172a] text-white rounded-md hover:bg-[#1e293b] transition-all"
      onClick={async () => {
        await handleAddStaffToProgram(programName, Array.from(staffIdsToAdd))
        setStaffIdsToAdd(new Set())
      }}
    >
      Add Selected ({staffIdsToAdd.size})
    </Button>
    <Button
      size="sm"
      variant="ghost"
      className="h-8 px-3 text-xs font-medium text-slate-500 hover:bg-white hover:text-slate-900 border border-transparent hover:border-slate-200 rounded-md transition-all"
      onClick={() => setStaffIdsToAdd(new Set())}
    >
      Clear
    </Button>
  </div>
)}
```

---

## Inline Delete Confirmation

Show "Confirm?" button after clicking delete/trash icon:

```tsx
const [staffIdPendingDelete, setStaffIdPendingDelete] = useState<string | null>(null)

// In render
{staffIdPendingDelete === staffId ? (
  <div className="flex items-center gap-1">
    <Button
      variant="destructive"
      size="sm"
      className="h-7 text-xs"
      onClick={() => {
        handleRemoveStaffFromProgram(programName, staffId)
        setStaffIdPendingDelete(null)
      }}
    >
      Confirm?
    </Button>
    <Button
      variant="ghost"
      size="sm"
      className="h-7 text-xs"
      onClick={() => setStaffIdPendingDelete(null)}
    >
      Cancel
    </Button>
  </div>
) : (
  <Button
    variant="ghost"
    size="icon"
    className="h-7 w-7 text-muted-foreground hover:text-destructive"
    onClick={() => setStaffIdPendingDelete(staffId)}
  >
    <Trash2 className="h-4 w-4" />
  </Button>
)}
```

---

## Ghost Button Usage

Use `variant="ghost"` for secondary actions, icons, and inline controls:

```tsx
// Icon buttons
<Button variant="ghost" size="icon" className="h-8 w-8">
  <Edit2 className="h-4 w-4" />
</Button>

// Text buttons
<Button variant="ghost" size="sm" onClick={handleCancel}>
  Cancel
</Button>

// Destructive ghost (on hover)
<Button variant="ghost" className="text-destructive hover:text-destructive">
  <Trash2 className="h-4 w-4" />
</Button>
```

### Ghost Button Variants

| Style | Use Case |
|-------|----------|
| Default ghost | Secondary actions, cancel, dismiss |
| Ghost + icon | Edit, delete, expand/collapse |
| Ghost + hover text color | Destructive actions |
| Ghost + border on hover | "Clear" actions, unstyled alternative |

---

## Border & Pane Design

### Card Styling

- Use `border` sparingly - prefer spacing and dividers
- When using Card: `className="p-4 border"` or `className="p-4 border-2"` for emphasis
- Remove outer Card wrappers from dashboard panels

### Form Panes

- No border on edit forms inside panels
- Use subtle background `bg-muted/30` for input areas
- Rounded corners: `rounded` or `rounded-md`

```tsx
// Input area
<div className="bg-muted/30 rounded p-3">
  {/* form fields */}
</div>

// No border edit form
<div className="space-y-4">
  {/* form fields without border */}
</div>
```

### Warning/Info Banners

```tsx
<div className="mb-3 p-3 bg-blue-50 border border-blue-200 rounded text-sm text-blue-800">
  <p className="font-medium mb-1">Label:</p>
  <p>Message content.</p>
</div>
```

---

## Typography & Spacing

### Text Hierarchy

| Element | Style |
|---------|-------|
| Page title | `text-xl font-semibold` |
| Section header | `text-lg font-semibold` |
| Subsection header | `text-sm font-medium` |
| Labels | `text-sm text-muted-foreground` |
| Helper text | `text-xs text-muted-foreground` |
| Uppercase labels | `text-xs font-semibold uppercase tracking-wide` |

### Spacing Scale

| Context | Value |
|---------|-------|
| Panel padding | `pt-6 space-y-6` |
| Section gap | `space-y-6` or `gap-6` |
| Component internal | `p-3`, `p-4` |
| Icon gaps | `gap-1`, `gap-2` |
| Inline spacing | `space-x-2`, `space-x-4` |

---

## Icons (Lucide React)

Always use lucide-react icons. Common patterns:

```tsx
import { 
  ChevronRight,    // collapsed section
  ChevronDown,     // expanded section
  ChevronUp,      // collapse upward
  Trash2,         // delete
  Edit2,          // edit
  Plus,           // add
  X,              // close/cancel
  Check,          // confirm
} from 'lucide-react'
```

---

## Quick Reference

### Common Component Patterns

```tsx
// Section with header + divider
<section>
  <h3 className="text-lg font-semibold mb-4">Section Title</h3>
  <hr className="border-border mb-4" />
  <div>Content</div>
</section>

// Collapsible group header
<div className="flex items-center gap-2 mb-3">
  <button onClick={toggle}>
    {isExpanded ? <ChevronDown /> : <ChevronRight />}
  </button>
  <span className="font-medium">GROUP NAME</span>
</div>

// Staff row with expand
<div className="flex items-center justify-between py-2">
  <div className="flex items-center gap-2">
    <button onClick={toggleExpand}>
      {isExpanded ? <ChevronDown /> : <ChevronRight />}
    </button>
    <span className="font-medium">{name}</span>
    <span className="text-xs text-muted-foreground">({rank})</span>
  </div>
  {pendingDelete ? (
    <ConfirmDeleteButtons />
  ) : (
    <GhostIconButton icon={<Trash2 />} onClick={() => setPendingDelete(id)} />
  )}
</div>
```

### Anti-Patterns to Avoid

- ❌ Nested Card > 2 levels
- ❌ Multiple border layers
- ❌ Native `<select>` elements
- ❌ Emoji as icons
- ❌ No confirmation for destructive actions
- ❌ All content visible (use progressive disclosure)
- ❌ Duplicate titles in panels

---
> Source: [alvin19921008-arch/RBIP-web-app](https://github.com/alvin19921008-arch/RBIP-web-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
