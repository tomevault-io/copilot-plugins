## grg-kit

> You are editing an Angular component. Before writing UI code:


# Angular Component Development with GRG Kit

You are editing an Angular component. Before writing UI code:

## Quick Reference

### Spartan-NG Components (Pre-installed)
These components are already available - just import and use them:
accordion, alert, alert-dialog, avatar, badge, breadcrumb, button, calendar, card, checkbox, collapsible, combobox, command, context-menu, data-table, date-picker, dialog, dropdown-menu, form-field, hover-card, input, label, menubar, navigation-menu, pagination, popover, progress, radio-group, scroll-area, select, separator, sheet, sidebar, skeleton, slider, sonner, spinner, switch, table, tabs, textarea, toggle, tooltip

### Import Patterns

**Spartan-NG (hlm prefix):**
```typescript
import { HlmButtonImports } from '@spartan-ng/helm/button';
import { HlmCardImports } from '@spartan-ng/helm/card';
import { BrnDialogImports } from '@spartan-ng/brain/dialog';
import { HlmDialogImports } from '@spartan-ng/helm/dialog';
```

**GRG Kit (grg- prefix):**
```typescript
import { GrgFileUploadImports } from '@grg-kit/ui/file-upload';
```

### Common Patterns

**Button:**
```html
<button hlmBtn>Default</button>
<button hlmBtn variant="outline">Outline</button>
<button hlmBtn variant="destructive">Destructive</button>
```

**Card:**
```html
<section hlmCard>
  <div hlmCardHeader>
    <h3 hlmCardTitle>Title</h3>
    <p hlmCardDescription>Description</p>
  </div>
  <div hlmCardContent>Content</div>
</section>
```

**Form Field:**
```html
<hlm-form-field>
  <input hlmInput [formControl]="control" placeholder="Email" />
  <hlm-error>Required</hlm-error>
</hlm-form-field>
```

**Dialog:**
```html
<hlm-dialog>
  <button brnDialogTrigger hlmBtn>Open</button>
  <hlm-dialog-content *brnDialogContent="let ctx">
    <hlm-dialog-header>
      <h3 hlmDialogTitle>Title</h3>
    </hlm-dialog-header>
    <!-- content -->
  </hlm-dialog-content>
</hlm-dialog>
```

### Icons
```typescript
import { NgIcon, provideIcons } from '@ng-icons/core';
import { lucideCheck, lucideX } from '@ng-icons/lucide';

@Component({
  providers: [provideIcons({ lucideCheck, lucideX })],
  template: `<ng-icon hlm name="lucideCheck" />`
})
```

## When to Use MCP

Use MCP only for:
- **Blocks** (auth, shell, settings) - `mcp2_search_ui_resources({ query: "auth" })`
- **Themes** - `mcp2_list_available_resources({ category: "themes" })`
- **GRG Kit components** (file-upload) - `mcp2_search_ui_resources({ query: "file-upload" })`

**Do NOT use MCP for Spartan-NG components** - they are already installed!

## TailwindCSS Rules (CRITICAL)

### NEVER use raw Tailwind colors:
```html
<!-- ❌ FORBIDDEN -->
<div class="text-green-600 bg-green-100">Success</div>
<div class="text-red-500">Error</div>
<div class="bg-yellow-50 border-yellow-200">Warning</div>
```

### ALWAYS use semantic color tokens:
```html
<!-- ✅ CORRECT -->
<div class="bg-primary text-primary-foreground">Primary</div>
<div class="bg-destructive text-destructive-foreground">Error</div>
<div class="text-muted-foreground">Secondary text</div>
<div class="bg-muted">Subtle background</div>
<div class="border-border">Standard border</div>
```

### Available semantic colors:
`background`, `foreground`, `card`, `card-foreground`, `popover`, `popover-foreground`,
`primary`, `primary-foreground`, `secondary`, `secondary-foreground`, `muted`, `muted-foreground`,
`accent`, `accent-foreground`, `destructive`, `destructive-foreground`, `border`, `input`, `ring`

### Need a new color (e.g., success, warning)?
1. Add CSS variable to `src/styles.css` with light AND dark mode values
2. Register in `@theme inline` block
3. Then use: `bg-success text-success-foreground`

### Typography - use standard scale:
```html
<!-- ✅ CORRECT -->
<p class="text-sm">Small</p>
<p class="text-base">Body</p>
<h2 class="text-xl font-semibold">Heading</h2>

<!-- ❌ WRONG - arbitrary sizes -->
<p class="text-[13px]">Don't do this</p>
```

## Remember
- Spartan-NG components are pre-installed - just import and use
- Follow existing patterns in the codebase
- Use TailwindCSS v4 for styling with SEMANTIC colors only
- Prefer signals for state management
- NEVER use raw colors like text-green-600, bg-yellow-100, etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Genesis-Research) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
