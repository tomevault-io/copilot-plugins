## ux-text

> This is the Chedr Apex System v5.1, a React component library with Tailwind CSS following Apple Human Interface Guidelines. Philosophy: "Apple Precision meets Google Utility for tax intelligence."

# Chedr Apex System v5.1 - Cursor Rules

## Project Context
This is the Chedr Apex System v5.1, a React component library with Tailwind CSS following Apple Human Interface Guidelines. Philosophy: "Apple Precision meets Google Utility for tax intelligence."

## Code Generation Rules

### Colors - ALWAYS use these exact hex values:

**Brand Palette (CHEDR Amber - Apple System Orange):**
- Primary: #FF9500
- Primary Hover: #E88600
- Primary Active: #CC7700
- Primary Light: #FFF8F0
- Primary Dark Mode: #FF9F0A
- Primary Hover Dark: #FFB340

**Neutral Palette (Refined Slate):**
- Text Primary: #0F172A
- Text Secondary: #475569
- Text Tertiary: #64748B
- Border: #E2E8F0
- Background: #FFFFFF
- Background Alt: #F8FAFC

**Apple System Colors (Official Palette):**
- systemOrange: #FF9500 (light) / #FF9F0A (dark) - Brand Primary
- systemRed: #FF3B30 (light) / #FF453A (dark) - Error
- systemGreen: #34C759 (light) / #30D158 (dark) - Success
- systemBlue: #007AFF (light) / #0A84FF (dark) - Info, Links
- systemYellow: #FFCC00 (light) / #FFD60A (dark) - Warning
- systemTeal: #5AC8FA (light) / #64D2FF (dark) - Accent
- systemIndigo: #5856D6 (light) / #5E5CE6 (dark) - Accent
- systemPurple: #AF52DE (light) / #BF5AF2 (dark) - Premium
- systemPink: #FF2D55 (light) / #FF375F (dark) - Highlight
- systemGray: #8E8E93 (light/dark) - Disabled

### Typography
- Font: Inter (sans), JetBrains Mono (mono)
- Weights: 300 (light), 400 (normal), 500 (medium), 600 (semibold), 700 (bold), 800 (extrabold)

### Spacing (8pt Grid)
Use multiples of 8: 8, 16, 24, 32, 48, 64, 96 (px)

### Border Radius
- sm: 6px - Subtle rounding for small elements
- base: 12px - Friendly but professional for inputs and cards
- lg: 18px - Soft and approachable for larger containers
- pill: 980px - Fully rounded for primary actions (buttons, badges)

### Shadows
- sm: shadow-sm
- default: shadow-md
- lg: shadow-lg

### Component Imports
Always import from '@/components':
```tsx
import { Button, Input, Card, Alert, Modal } from '@/components';
```

### Button Variants
- primary: bg-[#FF9500] hover:bg-[#E88600] text-white rounded-full (pill shape)
- secondary: bg-transparent border border-[#E2E8F0] text-[#0F172A] rounded-full
- ghost: bg-transparent hover:bg-[#FFF8F0] text-[#FF9500] rounded-full
- destructive: bg-[#FF3B30] hover:bg-[#DC2626] text-white rounded-full

### Button Sizes (44px minimum touch target per Apple HIG)
- sm: px-4 py-2 text-sm min-h-[36px]
- md: px-6 py-3 text-base min-h-[44px]
- lg: px-8 py-4 text-lg min-h-[52px]

### Form Input Styling
```
border border-[#E2E8F0] rounded-xl px-4 py-3
focus:border-[#FF9500] focus:ring-2 focus:ring-[#FF9500]/50
```

### Card Styling
```
bg-white rounded-xl shadow-md p-6
```

### Transitions
Always add: transition-colors duration-200

### Focus States (Brand Amber Ring)
Always include: focus:outline-none focus-visible:ring-2 focus-visible:ring-[#FF9500] focus-visible:ring-offset-2

### Disabled States
Always include: disabled:opacity-50 disabled:cursor-not-allowed

### Hover States
- Buttons: hover:bg-[variant-hover-color]
- Cards: hover:shadow-lg hover:-translate-y-0.5
- Links: hover:text-[#E88600]

### Z-Index Scale
- dropdown: z-[100]
- sticky: z-[200]
- modal: z-[300]
- popover: z-[400]
- toast: z-[500]
- tooltip: z-[600]

### Icon Colors (Apple System Colors)
- Default: text-[#0F172A] (light) / text-[#F1F5F9] (dark)
- Interactive: text-[#FF9500] (light) / text-[#FF9F0A] (dark) - systemOrange
- Success: text-[#34C759] (light) / text-[#30D158] (dark) - systemGreen
- Warning: text-[#FFCC00] (light) / text-[#FFD60A] (dark) - systemYellow
- Error: text-[#FF3B30] (light) / text-[#FF453A] (dark) - systemRed
- Info: text-[#007AFF] (light) / text-[#0A84FF] (dark) - systemBlue
- Disabled: text-[#8E8E93]/50 - systemGray

**Icon state rules:**
- Outline icons = default state
- Filled icons = selected/active state
- Hover = text-[#FF9500]

### Chart Colors (Apple System Colors)
- Primary: #FF9500 / #FF9F0A (systemOrange)
- Secondary: #007AFF / #0A84FF (systemBlue)
- Tertiary: #34C759 / #30D158 (systemGreen)
- Quaternary: #5AC8FA / #64D2FF (systemTeal)
- Quinary: #5856D6 / #5E5CE6 (systemIndigo)
- Negative: #FF3B30 / #FF453A (systemRed)

**Chart accessibility:**
- Never rely on color alone - use patterns/labels
- Test in grayscale
- Support "Differentiate Without Color" accessibility setting

### Navigation Patterns
**Tab Bars (3-5 tabs max):**
- Navigation only (not actions)
- Filled icons for selected, outline for unselected
- Selected tint: #FF9500
- Unselected tint: #64748B

**Sidebars:**
- Use outlined icons (not filled)
- Row height: 44px minimum
- Selected background: #FFF8F0 (light) / #334155 (dark)

### Loading States
- < 200ms: No indicator
- 200ms - 1s: Subtle spinner
- 1s - 10s: Spinner with label
- > 10s: Progress bar

## File Patterns

### Component Structure
```tsx
import React from 'react';

interface ComponentProps {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  className?: string;
  children: React.ReactNode;
}

export function Component({
  variant = 'primary',
  size = 'md',
  className = '',
  children,
}: ComponentProps) {
  return (
    <div className={`base-classes ${className}`}>
      {children}
    </div>
  );
}
```

### Class String Pattern
Always use template literals with trim and replace:
```tsx
className={`
  base-classes
  ${conditionalClass}
  ${className}
`.trim().replace(/\s+/g, ' ')}
```

## Accessibility Requirements (Apple HIG Compliance)
- 44px minimum touch targets for all interactive elements
- All interactive elements need focus-visible states with brand amber ring
- Images need alt text
- Form inputs need labels
- Buttons need aria-labels when icon-only
- Modals need aria-modal and aria-labelledby
- Support prefers-reduced-motion
- WCAG 2.1 AA contrast compliance (4.5:1 for text, 3:1 for UI)

## Don't Do
- Don't use inline styles
- Don't create new color values outside the defined palette
- Don't skip TypeScript types
- Don't forget transition classes
- Don't use arbitrary Tailwind values
- Don't use sharp corners on primary action buttons (use pill shape)
- Don't compromise 44px touch targets

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/belzlu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
