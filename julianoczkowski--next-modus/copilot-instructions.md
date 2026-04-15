## next-modus

> **Remember**: Styling violations are automatically caught by ESLint and pre-commit hooks. Always validate before completing tasks.


# Styling & Design System Standards

**Remember**: Styling violations are automatically caught by ESLint and pre-commit hooks. Always validate before completing tasks.

**CRITICAL**: All styling must use CSS custom properties from `@/app/globals.css` - no exceptions. Always run `npm run validate:styles` to verify compliance.

## CSS Custom Properties (MANDATORY)

### Color Management (CRITICAL)

```tsx
// ✅ REQUIRED - CSS custom properties only
className="text-primary bg-card border-border"
className="text-foreground bg-background hover:bg-muted"
style={{ color: 'hsl(var(--foreground))' }}

// ❌ FORBIDDEN - Hardcoded colors (ESLint will catch these)
className="text-blue-500 bg-red-400 border-green-300"
className="text-gray-900 bg-white border-gray-200"
style={{ color: '#333', backgroundColor: 'white' }}
```

### Available Color Variables

```css
/* Primary system colors */
--primary / --primary-foreground
--secondary / --secondary-foreground
--accent / --accent-foreground
--destructive / --destructive-foreground
--success / --success-foreground
--muted / --muted-foreground

/* Layout colors */
--background / --foreground
--card / --card-foreground
--border / --ring

/* Usage examples */
bg-primary text-primary-foreground
bg-card text-card-foreground border-border
bg-success text-success-foreground
text-muted-foreground
```

### Adding New Colors (MANDATORY PROCESS)

1. **Add to globals.css ONLY** - both `:root` and `.dark` sections
2. **Use HSL format**: `--new-color: hsl(210 40% 98%)`
3. **Test dark mode**: Ensure contrast works in both themes
4. **Validate**: Run `npm run validate:styles` to verify compliance

```css
/* @/app/globals.css - Example new color addition */
:root {
  --status-warning: hsl(38 92% 50%);
  --status-warning-foreground: hsl(48 96% 89%);
}

.dark {
  --status-warning: hsl(38 92% 50%);
  --status-warning-foreground: hsl(20 14% 4%);
}
```

## Modus Icons (ONLY ICON SYSTEM ALLOWED)

### Usage Pattern (MANDATORY)

```tsx
// ✅ REQUIRED - Modus icons only
<i className="modus-icons">settings</i>
<i className="modus-icons text-primary">add</i>
<i className="modus-icons mr-2">calendar</i>

// ❌ FORBIDDEN - Other icon libraries
import { Settings, User } from "lucide-react";     // NOT ALLOWED
import { FaUser } from "react-icons/fa";           // NOT ALLOWED
import { Settings } from "@heroicons/react/24/outline"; // NOT ALLOWED
```

### Critical Icon Validation

**ALWAYS** validate icon names against the official list before using them. Many icons fail to render due to name mismatches.

### Common Valid Icons

```tsx
// Navigation & UI
<i className="modus-icons">menu</i>
<i className="modus-icons">close</i>
<i className="modus-icons">arrow_back</i>
<i className="modus-icons">home</i>
<i className="modus-icons">dashboard</i>

// Actions & Controls
<i className="modus-icons">add</i>
<i className="modus-icons">edit_combination</i>
<i className="modus-icons">delete</i>
<i className="modus-icons">search</i>
<i className="modus-icons">settings</i>
<i className="modus-icons">save_disk</i>

// Status & Feedback
<i className="modus-icons">check_circle</i>
<i className="modus-icons">cancel_circle</i>
<i className="modus-icons">warning</i>
<i className="modus-icons">info</i>

// Content & Media
<i className="modus-icons">file</i>
<i className="modus-icons">folder_open</i>
<i className="modus-icons">image</i>
<i className="modus-icons">document</i>
```

### Icon Name Alternatives

```tsx
// Common mistakes - use these alternatives instead:
gear → settings
user → person
trash → delete
plus → add
minus → remove
edit → edit_combination
save → save_disk
```

### Icon Component Pattern

```tsx
import { cn } from "@/lib/utils";

interface ModusIconProps {
  name: string;
  size?: "sm" | "md" | "lg" | "xl";
  variant?: "default" | "primary" | "destructive" | "success" | "muted";
  className?: string;
}

export function ModusIcon({
  name,
  size = "md",
  variant = "default",
  className,
}: ModusIconProps) {
  return (
    <i
      className={cn(
        "modus-icons",
        {
          "text-sm": size === "sm",
          "text-base": size === "md",
          "text-lg": size === "lg",
          "text-xl": size === "xl",
        },
        {
          "text-foreground": variant === "default",
          "text-primary": variant === "primary",
          "text-destructive": variant === "destructive",
          "text-success": variant === "success",
          "text-muted-foreground": variant === "muted",
        },
        className
      )}
    >
      {name}
    </i>
  );
}

// Usage
<ModusIcon name="settings" size="lg" variant="primary" />
<ModusIcon name="check_circle" variant="success" />
<ModusIcon name="warning" variant="destructive" />
```

## Accessibility with Icons

### Proper ARIA Implementation

```tsx
// ✅ Icon-only buttons - include aria-label
<button aria-label="Delete item">
  <i className="modus-icons">delete</i>
</button>

// ✅ Decorative icons - mark as hidden
<button>
  <i className="modus-icons mr-2" aria-hidden="true">add</i>
  Add Item
</button>

// ❌ Icon without context
<button>
  <i className="modus-icons">delete</i>
</button>
```

## Layout & Spacing Standards

### Use CSS Custom Properties for Spacing

```tsx
// ✅ CSS custom properties
className="p-[var(--spacing)] gap-[var(--spacing)]"
className="rounded-[var(--radius)]"
style={{ padding: 'var(--spacing)', margin: 'var(--spacing-sm)' }}

// ❌ Hardcoded values
className="p-4 gap-4 rounded-lg"
style={{ padding: '16px', margin: '8px' }}
```

### Component Style Variants Pattern

```tsx
const styleVariants = {
  base: "p-[var(--spacing)] rounded-[var(--radius)] border shadow-[var(--shadow-sm)]",
  variant: {
    default: "bg-card text-card-foreground border-border",
    muted: "bg-muted text-muted-foreground border-transparent",
    primary: "bg-primary text-primary-foreground border-transparent",
    success: "bg-success text-success-foreground border-transparent",
  },
  size: {
    sm: "text-sm p-2",
    md: "text-base p-[var(--spacing)]",
    lg: "text-lg p-6",
  },
};

// Usage with cn()
<div
  className={cn(
    styleVariants.base,
    styleVariants.variant[variant],
    styleVariants.size[size],
    className
  )}
/>;
```

## Dark Mode Support

Dark mode is **automatic** when using CSS custom properties - no additional code needed.

```tsx
// ✅ Automatic dark mode support
<div className="bg-card text-card-foreground border-border">
  <i className="modus-icons text-primary">lightbulb_on</i>
  Content automatically adapts to theme
</div>
```

## Validation Requirements (MANDATORY)

### Before Task Completion

```bash
# CRITICAL - Run before considering any task complete
npm run validate:styles    # Comprehensive validation
npm run lint:colors       # Color-specific validation
npm run lint              # ESLint validation
```

### ESLint Rules Enforced

- Catches hardcoded Tailwind colors (`text-blue-500`, `bg-red-400`)
- Enforces CSS custom properties usage
- Validates Modus icon implementations
- Prevents inline style violations

### Pre-commit Hooks

Pre-commit hooks automatically prevent code with styling violations from being committed.

## Integration with JSX

### Quote Escaping (CRITICAL)

```tsx
// ✅ REQUIRED - Use HTML entities for quotes in JSX
<p>Welcome to the &ldquo;Settings&rdquo; page</p>
<span>User said &quot;Hello&quot;</span>

// ❌ FORBIDDEN - Raw quotes cause linting errors
<p>Welcome to the "Settings" page</p>
<span>User said "Hello"</span>
```

## Key Compliance Checklist

- [ ] Only CSS custom properties used (no hardcoded colors)
- [ ] Only Modus icons used (no other icon libraries)
- [ ] Icon names validated against official list
- [ ] Proper ARIA labels for accessibility
- [ ] Dark mode support through CSS custom properties
- [ ] Quotes properly escaped in JSX content
- [ ] Validation passes (`npm run validate:styles`)
- [ ] ESLint rules pass with no styling violations

- [ ] Validation passes (`npm run validate:styles`)
- [ ] ESLint rules pass with no styling violations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julianoczkowski)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/julianoczkowski)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
