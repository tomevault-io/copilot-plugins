## contract-tacit-v0

> When installing ShadCN components, always use pnpm (never npm or yarn):

# ShadCN Component Rules

## Component Installation & Usage

### Always Use pnpm for ShadCN
When installing ShadCN components, always use pnpm (never npm or yarn):
```bash
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add badge card calendar dropdown-menu input label popover select table
pnpm dlx shadcn@latest add dialog alert-dialog
```

### Available Components

The following ShadCN components are installed and available for use:

#### Form Components
- `Button` from `@/components/ui/button`
- `Input` from `@/components/ui/input`
- `Label` from `@/components/ui/label` 
- `Textarea` from `@/components/ui/textarea`
- `Select` from `@/components/ui/select`
- `Calendar` from `@/components/ui/calendar`
- `Popover` from `@/components/ui/popover`

#### Display Components
- `Badge` from `@/components/ui/badge`
- `Card` from `@/components/ui/card`
- `Table` from `@/components/ui/table`
- `DropdownMenu` from `@/components/ui/dropdown-menu`

#### Dialog Components
- `Dialog` from `@/components/ui/dialog`
- `AlertDialog` from `@/components/ui/alert-dialog`

#### Layout Components
- `Separator` from `@/components/ui/separator`

### Component Usage Standards

#### Import Pattern
Always use the `@/components/ui/` path pattern:
```tsx
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"
import { 
    AlertDialog,
    AlertDialogAction,
    AlertDialogCancel,
    AlertDialogContent,
    AlertDialogDescription,
    AlertDialogFooter,
    AlertDialogHeader,
    AlertDialogTitle,
    AlertDialogTrigger,
} from "@/components/ui/alert-dialog"
```

#### Styling with cn()
Always use the `cn()` utility for conditional styling:
```tsx
import { cn } from "@/lib/utils"

<Button className={cn("default-styles", error && "error-styles")} />
```

#### Form Components
For forms, combine ShadCN components with proper validation:
```tsx
<Label htmlFor="title">Title *</Label>
<Input
    id="title"
    value={data.title}
    onChange={(e) => setData('title', e.target.value)}
    className={errors.title ? 'border-red-300' : ''}
/>
{errors.title && <p className="text-red-600 text-sm mt-1">{errors.title}</p>}
```

#### DataTable Pattern
For data tables, use TanStack Table with ShadCN components:
```tsx
import { ColumnDef } from "@tanstack/react-table"
import { Button } from "@/components/ui/button"
import { DropdownMenu } from "@/components/ui/dropdown-menu"
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table"
```

#### Dialog Pattern
Use AlertDialog for destructive actions like deletion:
```tsx
<AlertDialog>
    <AlertDialogTrigger asChild>
        <Button variant="destructive">Delete</Button>
    </AlertDialogTrigger>
    <AlertDialogContent>
        <AlertDialogHeader>
            <AlertDialogTitle>Are you sure?</AlertDialogTitle>
            <AlertDialogDescription>
                This action cannot be undone.
            </AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
            <AlertDialogCancel>Cancel</AlertDialogCancel>
            <AlertDialogAction onClick={handleDelete}>Delete</AlertDialogAction>
        </AlertDialogFooter>
    </AlertDialogContent>
</AlertDialog>
```

### Required Dependencies

These packages are required for ShadCN components:
- `@tanstack/react-table` - For data tables
- `date-fns` - For date formatting with Calendar
- `react-day-picker` - Calendar component dependency
- `lucide-react` - Icon library
- `@radix-ui/react-dialog` - Dialog primitives
- `@radix-ui/react-alert-dialog` - Alert dialog primitives

### Component Customization
- Always extend component variants using `class-variance-authority`
- Use Tailwind CSS classes for styling
- Follow the design system color palette
- Maintain accessibility standards (ARIA labels, keyboard navigation)

### File Organization
```
resources/js/
├── components/
│   ├── ui/           # ShadCN components (auto-generated)
│   └── contracts/    # Custom business components
│       ├── columns.tsx
│       └── data-table.tsx
└── pages/
    └── Contracts/    # Page components using ShadCN
        ├── Index.tsx
        ├── Create.tsx
        └── Edit.tsx
```

**Never modify files in `components/ui/` manually - regenerate them with shadcn CLI instead.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MakFly)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MakFly)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
