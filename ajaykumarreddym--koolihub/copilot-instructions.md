## koolihub

> Figma Design Integration - Design-to-Code Guidelines


# Figma Design Integration Rules

## Overview
Guidelines for integrating Figma designs into KooliHub codebase using the Figma MCP tools and maintaining design-code consistency.

---

## 1. Figma MCP Tool Usage

### 1.1 Available Tools
```typescript
// Primary tools for design extraction
mcp_Figma_get_design_context   // Generate UI code from Figma node
mcp_Figma_get_screenshot       // Visual reference screenshot
mcp_Figma_get_metadata         // Structure overview (XML format)
mcp_Figma_get_variable_defs    // Design tokens extraction
mcp_Figma_get_code_connect_map // Component mapping
```

### 1.2 Extracting Design Context
```typescript
// When given a Figma URL like:
// https://figma.com/design/ABC123/MyProject?node-id=1-2

// Extract:
fileKey: 'ABC123'
nodeId: '1:2' (or '1-2')

// Call with:
get_design_context({
  fileKey: 'ABC123',
  nodeId: '1:2',
  clientFrameworks: 'react',
  clientLanguages: 'typescript,html,css'
})
```

### 1.3 Design Token Extraction
```typescript
// Get variable definitions
get_variable_defs({
  fileKey: 'ABC123',
  nodeId: '1:2'
})

// Response format:
{
  'color/primary': '#FFD700',
  'color/secondary': '#6B7280',
  'spacing/sm': '8px',
  'radius/md': '12px'
}
```

---

## 2. Design-to-Code Workflow

### 2.1 Standard Workflow
```
1. Get Figma URL from designer
2. Extract nodeId and fileKey from URL
3. Call get_design_context for code generation
4. Call get_variable_defs for design tokens
5. Map Figma tokens to existing design system
6. Implement using existing UI components
7. Validate against screenshot
```

### 2.2 Component Mapping Priority
```
Figma Component → KooliHub Equivalent
─────────────────────────────────────
Button          → @/components/ui/button
Card            → @/components/ui/card
Input           → @/components/ui/input
Select          → @/components/ui/select
Modal           → @/components/ui/dialog
Dropdown        → @/components/ui/dropdown-menu
Toast           → @/components/ui/toast
Badge           → @/components/ui/badge
Avatar          → @/components/ui/avatar
Tabs            → @/components/ui/tabs
```

---

## 3. Design Token Mapping

### 3.1 Color Mapping
```typescript
// Figma Variable → CSS Variable → Tailwind Class

// Map Figma colors to existing tokens
const figmaToTokenMap = {
  // Figma naming → KooliHub token
  'primary': '--primary',
  'primary-foreground': '--primary-foreground',
  'secondary': '--secondary',
  'background': '--background',
  'foreground': '--foreground',
  'muted': '--muted',
  'accent': '--accent',
  'destructive': '--destructive',
  'border': '--border',
  
  // Service-specific
  'grocery': 'grocery-500',
  'trips': 'trips-500',
  'handyman': 'handyman-500',
};
```

### 3.2 Spacing Mapping
```typescript
const spacingMap = {
  // Figma → Tailwind
  '4': 'p-1',    // 4px
  '8': 'p-2',    // 8px
  '12': 'p-3',   // 12px
  '16': 'p-4',   // 16px
  '20': 'p-5',   // 20px
  '24': 'p-6',   // 24px
  '32': 'p-8',   // 32px
  '40': 'p-10',  // 40px
  '48': 'p-12',  // 48px
  '64': 'p-16',  // 64px
};
```

### 3.3 Typography Mapping
```typescript
const typographyMap = {
  // Figma text style → Tailwind classes
  'Heading/H1': 'text-4xl font-bold',
  'Heading/H2': 'text-3xl font-bold',
  'Heading/H3': 'text-2xl font-semibold',
  'Heading/H4': 'text-xl font-semibold',
  'Body/Large': 'text-lg',
  'Body/Regular': 'text-base',
  'Body/Small': 'text-sm',
  'Caption': 'text-xs text-muted-foreground',
  'Label': 'text-sm font-medium',
};
```

---

## 4. Implementation Rules

### 4.1 Never Hardcode Design Values
```typescript
// ❌ BAD - Hardcoded values from Figma
<div style={{ padding: '16px', backgroundColor: '#FFD700' }}>

// ✅ GOOD - Use design tokens
<div className="p-4 bg-primary">

// ❌ BAD - Inline pixel values
<div className="text-[14px] leading-[20px]">

// ✅ GOOD - Use semantic classes
<div className="text-sm leading-normal">
```

### 4.2 Color Extraction Rules
```typescript
// When Figma provides hex colors, map to tokens:

// Primary brand colors → use CSS variables
'#FFD700' → 'bg-primary' (update --primary if needed)
'#10B981' → 'bg-grocery-500'
'#3B82F6' → 'bg-trips-500'
'#F97316' → 'bg-handyman-500'

// Grayscale → use standard palette
'#F9FAFB' → 'bg-gray-50'
'#F3F4F6' → 'bg-gray-100'
'#E5E7EB' → 'bg-gray-200'
'#6B7280' → 'text-gray-500'
'#374151' → 'text-gray-700'
'#111827' → 'text-gray-900'

// Add new tokens to global.css if not exists
```

### 4.3 Spacing Extraction Rules
```typescript
// Convert Figma auto-layout to Tailwind

// Gap → gap-{n}
gap: 8   → gap-2
gap: 16  → gap-4
gap: 24  → gap-6

// Padding → p-{n}, px-{n}, py-{n}
padding: 16 → p-4
paddingX: 24, paddingY: 16 → px-6 py-4

// Margin → m-{n}, mx-{n}, my-{n}
marginTop: 32 → mt-8
```

---

## 5. Component Generation Rules

### 5.1 From Figma Auto-Layout
```typescript
// Figma Auto Layout → Tailwind Flex/Grid

// Horizontal layout
direction: horizontal → flex flex-row
spacing: 16 → gap-4

// Vertical layout
direction: vertical → flex flex-col
spacing: 8 → gap-2

// Wrap
wrap: true → flex-wrap

// Alignment
primaryAxisAlignItems: center → justify-center
counterAxisAlignItems: center → items-center
primaryAxisAlignItems: spaceBetween → justify-between
```

### 5.2 Component Structure Template
```typescript
// Generated component should follow this structure:

import React from 'react';
import { cn } from '@/lib/utils';
// Import UI primitives
import { Card } from '@/components/ui/card';
import { Button } from '@/components/ui/button';

// Define props interface
interface ComponentNameProps {
  // Props from Figma variants
  variant?: 'default' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  className?: string;
  children?: React.ReactNode;
}

// Export component
export function ComponentName({
  variant = 'default',
  size = 'md',
  className,
  children,
}: ComponentNameProps) {
  return (
    <div className={cn(
      // Base styles
      'flex flex-col gap-4 p-4 rounded-lg',
      // Variant styles
      variant === 'secondary' && 'bg-secondary',
      // Size styles
      size === 'sm' && 'p-2 gap-2',
      size === 'lg' && 'p-6 gap-6',
      // Allow overrides
      className
    )}>
      {children}
    </div>
  );
}
```

---

## 6. Asset Handling

### 6.1 Image Assets from Figma
```typescript
// When Figma returns asset URLs:

// 1. Download assets to public/images/
// 2. Use consistent naming: feature-name-asset-type.webp
// 3. Reference in code:

// ✅ GOOD - Relative paths
<img src="/images/hero-banner.webp" alt="Hero" />

// ✅ BETTER - Constants file
import { IMAGES } from '@/lib/constants';
<img src={IMAGES.HERO_BANNER} alt="Hero" />
```

### 6.2 Icon Mapping
```typescript
// Map Figma icons to Lucide equivalents

const iconMap = {
  'icon/arrow-right': ArrowRight,
  'icon/arrow-left': ArrowLeft,
  'icon/check': Check,
  'icon/close': X,
  'icon/menu': Menu,
  'icon/search': Search,
  'icon/user': User,
  'icon/cart': ShoppingCart,
  'icon/heart': Heart,
  'icon/star': Star,
};

// Usage
import { ArrowRight } from 'lucide-react';
<ArrowRight className="h-5 w-5" />
```

---

## 7. Responsive Implementation

### 7.1 Figma Frame Sizes → Breakpoints
```typescript
// Common Figma frame widths:
375px  → Mobile (default)
768px  → Tablet (md:)
1024px → Laptop (lg:)
1440px → Desktop (xl:)

// Implementation:
<div className="
  p-4 md:p-6 lg:p-8           // Progressive padding
  grid-cols-1 md:grid-cols-2 lg:grid-cols-4  // Responsive grid
  text-sm md:text-base         // Responsive text
">
```

### 7.2 Mobile-First from Figma
```typescript
// Start with mobile frame, enhance for larger
// Figma mobile design → base classes
// Figma desktop design → md: and lg: modifiers

// Example:
// Mobile: 1 column, small padding
// Desktop: 3 columns, large padding

<div className="
  grid grid-cols-1 gap-4 p-4
  md:grid-cols-2 md:gap-6
  lg:grid-cols-3 lg:p-8
">
```

---

## 8. Design System Sync

### 8.1 Token Update Process
```
1. Extract new tokens from Figma
2. Check if token exists in global.css
3. If new: Add to global.css and tailwind.config.ts
4. If changed: Update value (with design team approval)
5. Document change in design-tokens.mdc
```

### 8.2 Component Library Sync
```
1. New component from Figma → Check if similar exists
2. If exists: Extend with new variant
3. If new: Create in appropriate location
4. Update component documentation
5. Add Storybook/usage examples
```

---

## 9. Quality Checklist

### Before Implementation
- [ ] Received Figma URL with correct permissions
- [ ] Extracted nodeId and fileKey correctly
- [ ] Reviewed design tokens and mapped to existing
- [ ] Identified reusable components
- [ ] Checked responsive requirements

### After Implementation
- [ ] No hardcoded values (colors, spacing, fonts)
- [ ] Using existing UI components
- [ ] Responsive design implemented
- [ ] Dark mode compatible (if applicable)
- [ ] Accessibility attributes added
- [ ] Screenshot comparison passes

---

## 10. Common Patterns

### Card Component from Figma
```typescript
// Figma card with image, title, description, CTA

import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';

export function ProductCard({ product }: { product: Product }) {
  return (
    <Card className="overflow-hidden hover:shadow-md transition-shadow">
      <div className="aspect-square relative">
        <img 
          src={product.image} 
          alt={product.name}
          className="object-cover w-full h-full"
        />
      </div>
      <CardHeader className="p-4">
        <CardTitle className="text-lg line-clamp-1">{product.name}</CardTitle>
        <p className="text-sm text-muted-foreground line-clamp-2">
          {product.description}
        </p>
      </CardHeader>
      <CardContent className="p-4 pt-0">
        <div className="flex items-center justify-between">
          <span className="text-lg font-bold">₹{product.price}</span>
          <Button size="sm">Add to Cart</Button>
        </div>
      </CardContent>
    </Card>
  );
}
```

### Form Layout from Figma
```typescript
// Figma form with labels, inputs, validation

import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Button } from '@/components/ui/button';

export function ContactForm() {
  return (
    <form className="space-y-4">
      <div className="space-y-2">
        <Label htmlFor="name">Name</Label>
        <Input id="name" placeholder="Enter your name" />
      </div>
      
      <div className="space-y-2">
        <Label htmlFor="email">Email</Label>
        <Input id="email" type="email" placeholder="you@example.com" />
      </div>
      
      <Button type="submit" className="w-full">
        Submit
      </Button>
    </form>
  );
}
```

---

**Related Rules**:
- [design-tokens.mdc](./02-design-tokens.mdc) - Complete token system
- [color-system.mdc](./07-color-system.mdc) - Color mapping details
- [typography-strings.mdc](./03-typography-strings.mdc) - Typography system

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ajaykumarreddym) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
