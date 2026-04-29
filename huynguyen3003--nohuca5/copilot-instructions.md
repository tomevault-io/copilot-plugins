## nohuca5

> Always use CSS variables from [src/app/globals.css](mdc:src/app/globals.css) via Tailwind classes:

# Tailwind CSS Rules

## Design Token Usage

Always use CSS variables from [src/app/globals.css](mdc:src/app/globals.css) via Tailwind classes:

**✅ Correct Usage:**

```tsx
className = "bg-primary text-primary-foreground";
className = "border-border rounded-md";
className = "text-muted-foreground p-4";
className = "shadow-gaming";
```

**❌ Avoid:**

```tsx
className = "bg-[#ff9c0d]";
className = "text-[#ffffff]";
className = "rounded-[8px]";
```

## Gaming Theme Classes

Use gaming-specific classes for the app's aesthetic:

```tsx
// Backgrounds
className = "bg-gaming-bg"; // Main gaming background
className = "bg-gaming-card"; // Card backgrounds
className = "bg-gaming-dark"; // Very dark elements

// Accents
className = "bg-gaming-gold"; // Orange/gold accent
className = "border-gaming-border"; // Subtle orange borders

// Shadows
className = "shadow-gaming"; // Orange glow effect
```

## Spacing System

Use the predefined spacing scale:

- `p-1`, `p-2`, `p-4`, `p-6`, `p-8`, `p-12`, `p-16`
- `gap-1`, `gap-2`, `gap-4`, `gap-6`, `gap-8`, `gap-12`, `gap-16`
- `m-1`, `m-2`, `m-4`, `m-6`, `m-8`, `m-12`, `m-16`

## Border Radius

Use consistent border radius:

- `rounded-sm` (0.25rem)
- `rounded-md` (0.5rem)
- `rounded-lg` (0.75rem)
- `rounded-xl` (1rem)

## Responsive Design

Always include responsive variants:

```tsx
className = "p-4 md:p-6 lg:p-8";
className = "text-sm md:text-base lg:text-lg";
className = "grid-cols-1 md:grid-cols-2 lg:grid-cols-3";
```

## Class Organization

Use the `cn()` utility for conditional classes:

```tsx
import { cn } from "@/lib/cn";

className={cn(
  "base-classes",
  variant === "primary" && "bg-primary",
  size === "lg" && "p-6",
  className
)}
```

description:
globs:
alwaysApply: true

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HuyNguyen3003) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
