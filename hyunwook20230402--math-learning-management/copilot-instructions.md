## styling

> - **Primary**: Use shadcn/ui components from `src/components/ui/`

# 🎨 Styling & UI Guidelines

## Design System

### Component Library
- **Primary**: Use shadcn/ui components from `src/components/ui/`
- **Icons**: Use Lucide React icons (`lucide-react`)
- **Styling**: Tailwind CSS utility classes
- **Theme**: Support both light and dark modes

### Color System
Use CSS custom properties for consistent theming:
```css
--primary: hsl(var(--primary))
--secondary: hsl(var(--secondary))
--destructive: hsl(var(--destructive))
--muted: hsl(var(--muted))
```

### Component Styling Rules

#### Button Variants
```typescript
<Button variant="default">Primary Action</Button>
<Button variant="secondary">Secondary Action</Button>
<Button variant="destructive">Delete Action</Button>
<Button variant="outline">Cancel Action</Button>
<Button variant="ghost">Subtle Action</Button>
```

#### Card Components
```typescript
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>
    {/* Content */}
  </CardContent>
</Card>
```

#### Form Components
```typescript
<div className="space-y-4">
  <div>
    <Label htmlFor="field">Field Label</Label>
    <Input id="field" placeholder="Placeholder" />
  </div>
</div>
```

## Layout Guidelines

### Page Structure
```typescript
<div className="min-h-screen bg-background">
  {profile && <Header />}
  <main className="container mx-auto px-4 py-8">
    {/* Page content */}
  </main>
</div>
```

### Responsive Design
- **Mobile First**: Design for mobile, then enhance for larger screens
- **Breakpoints**: 
  - Mobile: `< 768px`
  - Tablet: `768px - 1024px`
  - Desktop: `> 1024px`

### Grid Systems
```typescript
// 2-column grid on desktop, 1-column on mobile
<div className="grid grid-cols-1 md:grid-cols-2 gap-4">

// 3-column grid with responsive breakpoints
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
```

## Spacing & Typography

### Spacing Scale
- Use Tailwind's spacing scale: `space-y-4`, `gap-6`, `p-4`, `m-2`
- Consistent spacing: `space-y-4` for vertical, `gap-4` for grid/flex

### Typography
```typescript
<h1 className="text-3xl font-bold">Page Title</h1>
<h2 className="text-2xl font-semibold">Section Title</h2>
<h3 className="text-xl font-medium">Subsection Title</h3>
<p className="text-muted-foreground">Description text</p>
```

## Interactive Elements

### Loading States
```typescript
{loading ? (
  <div className="flex items-center justify-center p-8">
    <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary"></div>
  </div>
) : (
  // Content
)}
```

### Empty States
```typescript
<div className="text-center py-8 text-muted-foreground">
  <FileText className="h-12 w-12 mx-auto mb-4" />
  <p>No items found</p>
  <p className="text-sm mt-2">Try adjusting your search criteria</p>
</div>
```

### Error States
```typescript
<div className="rounded-lg border border-destructive/50 bg-destructive/10 p-4">
  <p className="text-destructive font-medium">Error occurred</p>
  <p className="text-destructive/80 text-sm mt-1">Error details</p>
</div>
```

## Animation & Transitions

### Hover Effects
```typescript
className="hover:bg-muted/50 transition-colors"
className="hover:scale-105 transition-transform"
```

### Loading Animations
```typescript
className="animate-spin" // For spinners
className="animate-pulse" // For skeleton loading
```

## Accessibility

### Focus Management
- Ensure all interactive elements are keyboard accessible
- Use proper focus indicators
- Implement skip links for navigation

### ARIA Labels
```typescript
<Button aria-label="Delete item">
  <Trash className="h-4 w-4" />
</Button>
```

### Color Contrast
- Ensure sufficient contrast ratios
- Don't rely solely on color to convey information
- Test with color blindness simulators

---
> Source: [hyunwook20230402/math-learning-management](https://github.com/hyunwook20230402/math-learning-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
