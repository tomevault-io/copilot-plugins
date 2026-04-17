## template

> enableSystem={false}


# Axolutions Design System Rules

## Overview
This project uses the Axolutions Design System based on shadcn/ui "New York" style with a custom purple theme (#5D0EC1), OKLCH color space, Public Sans font, and 0.75rem base border radius.

## Core Configuration

### Technology Stack
- Next.js with App Router
- TypeScript
- Tailwind CSS v4
- shadcn/ui (New York style)
- Public Sans font family
- OKLCH color space

### Initial Setup Commands
```bash
npx create-next-app@latest --typescript --tailwind --app
npx shadcn@latest init
# Choose: New York style, Neutral base color, CSS variables: Yes
```

## Color System

### Primary Brand Color
- Axolutions Purple: `#5D0EC1`
- OKLCH: `oklch(0.45 0.25 290)`
- Usage: Primary buttons, links, main actions

### Color Tokens (Light Mode)
```css
--background: oklch(0.99 0.005 290);
--foreground: oklch(0.25 0.02 290);
--primary: oklch(0.45 0.25 290);
--primary-foreground: oklch(1 0 0);
--secondary: oklch(0.96 0.03 290);
--muted: oklch(0.97 0.01 290);
--accent: oklch(0.92 0.08 290);
--destructive: oklch(0.577 0.245 27.325);
--border: oklch(0.90 0.02 290);
--input: oklch(0.95 0.01 290);
--ring: oklch(0.45 0.25 290);
```

### Color Tokens (Dark Mode)
```css
--background: oklch(0.145 0 0);
--foreground: oklch(0.985 0 0);
--primary: oklch(0.45 0.25 290);
--card: oklch(0.145 0 0);
--muted: oklch(0.269 0 0);
--border: oklch(0.269 0 0);
```

### Color Usage Guidelines
- Primary actions: Use `className="bg-primary hover:bg-primary/90"`
- Secondary actions: Use `variant="secondary"`
- Destructive actions: Use `variant="destructive"`
- Subtle backgrounds: Use `bg-muted` or `bg-secondary`
- Text hierarchy: `text-foreground`, `text-muted-foreground`

## Typography

### Font Configuration
```typescript
import { Public_Sans } from "next/font/google"

const publicSans = Public_Sans({
  subsets: ["latin"],
  variable: "--font-sans",
  weight: ["300", "400", "500", "600", "700"],
})

// In layout:
<body className={publicSans.className}>
```

### Typography Scale
```tsx
// Headings
<h1 className="text-4xl font-bold">Main Title</h1>
<h2 className="text-3xl font-semibold">Section Title</h2>
<h3 className="text-2xl font-semibold">Subsection Title</h3>
<h4 className="text-xl font-medium">Card Title</h4>

// Body text
<p className="text-base">Standard text</p>
<p className="text-sm text-muted-foreground">Secondary text</p>
<p className="text-xs text-muted-foreground">Small text</p>

// Font weights
font-light (300)
font-normal (400)
font-medium (500)
font-semibold (600)
font-bold (700)
```

## Spacing System

### Spacing Scale (based on Tailwind 4px units)
```tsx
// Padding
p-2  // 8px - Very small
p-3  // 12px - Small
p-4  // 16px - Standard
p-6  // 24px - Medium
p-8  // 32px - Large

// Gaps
gap-2  // 8px - Between small elements
gap-3  // 12px - Between related elements
gap-4  // 16px - Between group elements
gap-6  // 24px - Between sections

// Margins
mb-4  // 16px - Between paragraphs
mb-6  // 24px - Between sections
mb-8  // 32px - Between large blocks
```

### Layout Patterns
```tsx
// Main container
<div className="container mx-auto px-4 py-8">

// Responsive grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

// Standard card
<Card className="p-6">
  <CardHeader className="px-0 pt-0">
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent className="px-0 pb-0">
    Content
  </CardContent>
</Card>
```

## Border Radius

### Base Value
- Base radius: `0.75rem` (12px)

### Radius Variants
```tsx
rounded-sm    // 8px  (base - 4px)
rounded-md    // 10px (base - 2px)
rounded-lg    // 12px (base) - DEFAULT
rounded-xl    // 16px (base + 4px)
rounded-2xl   // 20px (base + 8px)
rounded-full  // Circular
```

### Usage by Component
- Buttons: `rounded-lg` (12px)
- Cards: `rounded-lg` (12px)
- Inputs: `rounded-md` (10px)
- Badges: `rounded-md` or `rounded-full`
- Avatars: `rounded-full`
- Modals: `rounded-xl` (16px)

## Component Guidelines

### Button Variants
```tsx
// Primary action
<Button className="bg-primary hover:bg-primary/90 rounded-lg">
  Primary Action
</Button>

// Secondary action
<Button variant="secondary" className="rounded-lg">
  Secondary Action
</Button>

// Outline
<Button variant="outline" className="rounded-lg">
  Cancel
</Button>

// Destructive
<Button variant="destructive" className="rounded-lg">
  Delete
</Button>
```

### Card Component
```tsx
<Card className="rounded-lg shadow-md hover:shadow-lg transition-shadow">
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>
    Content
  </CardContent>
  <CardFooter>
    Actions
  </CardFooter>
</Card>
```

### Input Component
```tsx
<div className="space-y-2">
  <Label htmlFor="email">Email</Label>
  <Input
    id="email"
    type="email"
    placeholder="your@email.com"
    className="rounded-md"
  />
</div>
```

## Theme Configuration

### globals.css Structure
```css
@import "tailwindcss";

@custom-variant dark (&:is(.dark *));

:root {
  /* Light mode variables */
}

.dark {
  /* Dark mode variables */
}

@theme inline {
  /* Tailwind theme customization */
  --font-sans: "Public Sans", system-ui, sans-serif;
  --radius-lg: var(--radius);
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

### Theme Provider Setup
```tsx
import { ThemeProvider } from "@/components/theme-provider"

<ThemeProvider
  attribute="class"
  defaultTheme="light"
  enableSystem={false}
  disableTransitionOnChange
>
  {children}
</ThemeProvider>
```

## shadcn/ui Components

### Installation Commands
```bash
# Essential components
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add input
npx shadcn@latest add label
npx shadcn@latest add select
npx shadcn@latest add dialog
npx shadcn@latest add dropdown-menu
npx shadcn@latest add table
npx shadcn@latest add badge
npx shadcn@latest add avatar
npx shadcn@latest add separator
npx shadcn@latest add toast
npx shadcn@latest add sonner
```

## Responsive Design

### Breakpoints (Tailwind defaults)
```tsx
sm:   // 640px
md:   // 768px
lg:   // 1024px
xl:   // 1280px
2xl:  // 1536px

// Usage example
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
```

## Accessibility

### Focus States
All interactive elements must have visible focus states using the `ring` color:
```tsx
<Button className="focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2">
```

### Color Contrast
Ensure all text meets WCAG AA standards:
- Normal text: 4.5:1 contrast ratio
- Large text: 3:1 contrast ratio
- Use `text-foreground` on `bg-background` for optimal contrast

## Best Practices

### DO's
- Use semantic HTML elements
- Implement proper color contrast
- Use the spacing system consistently
- Apply border radius according to component type
- Leverage Tailwind utility classes
- Keep components responsive
- Use OKLCH color space for custom colors
- Apply proper focus states

### DON'Ts
- Don't use arbitrary values without justification
- Don't mix color systems (stick to OKLCH)
- Don't use inline styles
- Don't ignore the spacing scale
- Don't use non-standard border radius values
- Don't skip accessibility considerations
- Don't use hex colors directly in code

## Component Composition Example

```tsx
export default function DashboardCard() {
  return (
    <Card className="rounded-lg shadow-md hover:shadow-lg transition-shadow">
      <CardHeader className="space-y-1">
        <CardTitle className="text-2xl font-semibold">
          Dashboard Overview
        </CardTitle>
        <CardDescription className="text-sm text-muted-foreground">
          Your activity summary
        </CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div className="p-4 bg-muted rounded-lg">
            <p className="text-sm text-muted-foreground">Total Users</p>
            <p className="text-2xl font-bold">1,234</p>
          </div>
          <div className="p-4 bg-muted rounded-lg">
            <p className="text-sm text-muted-foreground">Active Sessions</p>
            <p className="text-2xl font-bold">567</p>
          </div>
        </div>
      </CardContent>
      <CardFooter>
        <Button className="w-full bg-primary hover:bg-primary/90 rounded-lg">
          View Details
        </Button>
      </CardFooter>
    </Card>
  )
}
```

## Maintenance

### Updating Components
```bash
# Check for updates
npx shadcn@latest diff

# Update specific component
npx shadcn@latest add button --overwrite
```

### Adding Custom Colors
```css
/* In globals.css :root and .dark */
--success: oklch(0.65 0.20 145);
--warning: oklch(0.75 0.15 85);
--info: oklch(0.60 0.18 250);

/* In @theme inline */
--color-success: var(--success);
--color-warning: var(--warning);
--color-info: var(--info);
```

## Project Structure

```
src/
├── app/
│   ├── globals.css          # Design system CSS
│   └── layout.tsx           # Font and theme configuration
├── components/
│   ├── ui/                  # shadcn components
│   └── theme-provider.tsx   # Theme management
└── lib/
    └── utils.ts             # Utility functions (cn)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gustavo-Alves-Araujo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
