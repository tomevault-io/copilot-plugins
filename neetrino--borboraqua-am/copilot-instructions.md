## borboraqua-am

> Figma-ի և դիզայնի աշխատանքի կանոններ։ Կիրառվում է դիզայնը կոդի թարգմանելիս։


# FIGMA ԵՎ ԴԻԶԱՅՆԻ ԿԱՆՈՆՆԵՐ

> Դիզայնի ճշգրիտ թարգմանություն կոդ։ Design System First։

---

## 🎯 ՀԻՄՆԱԿԱՆ ՍԿԶԲՈՒՆՔՆԵՐ

1. **Pixel Perfect** — ճշգրիտ համապատասխանություն մակետին
2. **Design Tokens** — գույներ, բացատներ, ֆոնտեր token-ներից
3. **Կոմպոնենտային մոտեցում** — վերաօգտագործվող UI էլեմենտներ
4. **Responsive** — ադապտիվություն բոլոր սարքերում

---

## 🔗 FIGMA MCP-ՈՒՄ ԱՇԽԱՏԱՆՔ

### Figma-ից տվյալների ստացում

```
Դիզայնի հետ աշխատելիս.

1. ՍՏԱԳՆԻ՛ր Figma ֆրեյմ/կոմպոնենտի հղումը օգտատիրոջից
2. ՕԳՏԱԳՈՐԶԻ՛Ր Figma MCP կառուցվածքը ստանալու համար
3. ԱՌԱՋԻՆԻ՛ր.
   - Էլեմենտների չափեր
   - Բացատներ և gaps
   - Գույներ (վերածի՛ր token-ների)
   - Տիպոգրաֆիկա
   - Վիճակներ (hover, active, disabled)
4. ՍՏԵՂԾԻ՛ր կոմպոնենտ Design System-ի կանոններին համապատասխան
```

### Figma կոմպոնենտի վերլուծություն

```markdown
## Կոմպոնենտի վերլուծություն. [Անուն]

**Կառուցվածք.**
- Կոնտեյներ. [չափեր, padding, border-radius]
- Էլեմենտներ. [ներդրված էլեմենտների ցանկ]

**Սթայլեր.**
- Background. [գույն → token]
- Text. [գույն, չափ, weight → token-ներ]
- Border. [գույն, հաստություն → token-ներ]
- Shadow. [պարամետրներ → token]

**Վիճակներ.**
- Default. [սթայլեր]
- Hover. [փոփոխություններ]
- Active. [փոփոխություններ]
- Disabled. [փոփոխություններ]

**Responsive.**
- Desktop. [սթայլեր]
- Tablet. [փոփոխություններ]
- Mobile. [փոփոխություններ]
```

---

## 🎨 DESIGN TOKENS

### Թոքենների կառուցվածք

```typescript
// tokens/colors.ts
export const colors = {
  // Brand
  primary: {
    50: '#eff6ff',
    100: '#dbeafe',
    500: '#3b82f6',
    600: '#2563eb',
    700: '#1d4ed8',
  },
  secondary: {
    // ...
  },
  
  // Semantic
  success: '#10b981',
  warning: '#f59e0b',
  error: '#ef4444',
  info: '#3b82f6',
  
  // Neutral
  gray: {
    50: '#f9fafb',
    100: '#f3f4f6',
    200: '#e5e7eb',
    300: '#d1d5db',
    400: '#9ca3af',
    500: '#6b7280',
    600: '#4b5563',
    700: '#374151',
    800: '#1f2937',
    900: '#111827',
  },
  
  // Backgrounds
  background: {
    primary: '#ffffff',
    secondary: '#f9fafb',
    tertiary: '#f3f4f6',
  },
  
  // Text
  text: {
    primary: '#111827',
    secondary: '#4b5563',
    tertiary: '#9ca3af',
    inverse: '#ffffff',
  },
} as const;

// tokens/spacing.ts
export const spacing = {
  0: '0',
  1: '0.25rem',   // 4px
  2: '0.5rem',    // 8px
  3: '0.75rem',   // 12px
  4: '1rem',      // 16px
  5: '1.25rem',   // 20px
  6: '1.5rem',    // 24px
  8: '2rem',      // 32px
  10: '2.5rem',   // 40px
  12: '3rem',     // 48px
  16: '4rem',     // 64px
  20: '5rem',     // 80px
  24: '6rem',     // 96px
} as const;

// tokens/typography.ts
export const typography = {
  fontFamily: {
    sans: ['Inter', 'system-ui', 'sans-serif'],
    mono: ['JetBrains Mono', 'monospace'],
  },
  fontSize: {
    xs: ['0.75rem', { lineHeight: '1rem' }],
    sm: ['0.875rem', { lineHeight: '1.25rem' }],
    base: ['1rem', { lineHeight: '1.5rem' }],
    lg: ['1.125rem', { lineHeight: '1.75rem' }],
    xl: ['1.25rem', { lineHeight: '1.75rem' }],
    '2xl': ['1.5rem', { lineHeight: '2rem' }],
    '3xl': ['1.875rem', { lineHeight: '2.25rem' }],
    '4xl': ['2.25rem', { lineHeight: '2.5rem' }],
  },
  fontWeight: {
    normal: '400',
    medium: '500',
    semibold: '600',
    bold: '700',
  },
} as const;

// tokens/shadows.ts
export const shadows = {
  sm: '0 1px 2px 0 rgb(0 0 0 / 0.05)',
  md: '0 4px 6px -1px rgb(0 0 0 / 0.1)',
  lg: '0 10px 15px -3px rgb(0 0 0 / 0.1)',
  xl: '0 20px 25px -5px rgb(0 0 0 / 0.1)',
} as const;

// tokens/radius.ts
export const radius = {
  none: '0',
  sm: '0.25rem',    // 4px
  md: '0.375rem',   // 6px
  lg: '0.5rem',     // 8px
  xl: '0.75rem',    // 12px
  '2xl': '1rem',    // 16px
  full: '9999px',
} as const;
```

### Tailwind կոնֆիգուրացիա

```typescript
// tailwind.config.ts
import { colors, spacing, typography, shadows, radius } from './tokens';

export default {
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: colors.primary,
        secondary: colors.secondary,
        success: colors.success,
        warning: colors.warning,
        error: colors.error,
        gray: colors.gray,
      },
      fontFamily: typography.fontFamily,
      fontSize: typography.fontSize,
      fontWeight: typography.fontWeight,
      spacing: spacing,
      boxShadow: shadows,
      borderRadius: radius,
    },
  },
};
```

---

## 🧩 UI ԿՈՄՊՈՆԵՆՏՆԵՐ

### Ստեղծման կանոններ

```tsx
// ✅ ՃԻՇՏ — կոմպոնենտ տարբերակներով
import { cva, type VariantProps } from 'class-variance-authority';
import { clsx } from 'clsx';

const buttonVariants = cva(
  // Հիմնական սթայլեր (ընդհանուր բոլորի համար)
  [
    'inline-flex items-center justify-center',
    'font-medium transition-colors',
    'focus-visible:outline-none focus-visible:ring-2',
    'disabled:opacity-50 disabled:pointer-events-none',
  ],
  {
    variants: {
      variant: {
        primary: [
          'bg-primary-600 text-white',
          'hover:bg-primary-700',
          'focus-visible:ring-primary-500',
        ],
        secondary: [
          'bg-gray-100 text-gray-900',
          'hover:bg-gray-200',
          'focus-visible:ring-gray-500',
        ],
        outline: [
          'border-2 border-primary-600 text-primary-600',
          'hover:bg-primary-50',
          'focus-visible:ring-primary-500',
        ],
        ghost: [
          'text-gray-600',
          'hover:bg-gray-100',
          'focus-visible:ring-gray-500',
        ],
        danger: [
          'bg-error text-white',
          'hover:bg-red-600',
          'focus-visible:ring-red-500',
        ],
      },
      size: {
        sm: 'h-8 px-3 text-sm rounded-md',
        md: 'h-10 px-4 text-base rounded-lg',
        lg: 'h-12 px-6 text-lg rounded-lg',
        icon: 'h-10 w-10 rounded-lg',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean;
}

export function Button({
  variant,
  size,
  isLoading,
  className,
  children,
  disabled,
  ...props
}: ButtonProps) {
  return (
    <button
      className={clsx(buttonVariants({ variant, size }), className)}
      disabled={disabled || isLoading}
      {...props}
    >
      {isLoading ? (
        <>
          <Spinner className="mr-2 h-4 w-4" />
          Loading...
        </>
      ) : (
        children
      )}
    </button>
  );
}
```

### Կոմպոնենտների անվանավորում

```
UI կոմպոնենտներ (հիմնական):
├── Button.tsx
├── Input.tsx
├── Select.tsx
├── Checkbox.tsx
├── Radio.tsx
├── Switch.tsx
├── Modal.tsx
├── Dialog.tsx
├── Dropdown.tsx
├── Tooltip.tsx
├── Badge.tsx
├── Avatar.tsx
├── Card.tsx
├── Spinner.tsx
└── Skeleton.tsx

Բաղադրյալ կոմպոնենտներ:
├── Form/
│   ├── FormField.tsx
│   ├── FormLabel.tsx
│   ├── FormError.tsx
│   └── index.tsx
├── Table/
│   ├── Table.tsx
│   ├── TableHead.tsx
│   ├── TableRow.tsx
│   ├── TableCell.tsx
│   └── index.tsx
└── ...
```

---

## 📐 FIGMA → ԿՈԴ ԹԱՐԳՄԱՆՈՒԹՅՈՒՆ

### Figma գույներ → Թոքեններ

```typescript
// Figma-ն օգտագործում է hex կամ rgba
// Փոխարկի՛ր մոտակա թոքենին

// Figma: #3B82F6 → primary-500
// Figma: rgba(59, 130, 246, 0.1) → primary-500/10

// Map-ավորում
const figmaToToken: Record<string, string> = {
  '#3B82F6': 'primary-500',
  '#2563EB': 'primary-600',
  '#EF4444': 'error',
  '#10B981': 'success',
  // ...
};
```

### Figma չափեր → Tailwind

```typescript
// Figma: 16px padding → p-4
// Figma: 24px margin → m-6
// Figma: 8px gap → gap-2

// Ստանդարտ սանդղակ
// 4px = 1 (rem: 0.25)
// 8px = 2
// 12px = 3
// 16px = 4
// 20px = 5
// 24px = 6
// 32px = 8
// 40px = 10
// 48px = 12
```

### Figma տիպոգրաֆիկա → Tailwind

```typescript
// Figma Font:
// - Family: Inter
// - Size: 16px
// - Weight: 500
// - Line Height: 24px

// Tailwind:
// font-sans text-base font-medium leading-6

// Չափերի map-ավորում
// 12px → text-xs
// 14px → text-sm
// 16px → text-base
// 18px → text-lg
// 20px → text-xl
// 24px → text-2xl
// 30px → text-3xl
// 36px → text-4xl
```

---

## 📱 RESPONSIVE ԴԻԶԱՅՆ

### Breakpoints

```typescript
// Tailwind breakpoints
// sm: 640px
// md: 768px
// lg: 1024px
// xl: 1280px
// 2xl: 1536px

// Mobile First!
<div className="
  p-4          // Mobile: 16px padding
  md:p-6       // Tablet: 24px padding
  lg:p-8       // Desktop: 32px padding
">
```

### Responsive կոմպոնենտ

```tsx
// Ադապտիվ ցանց
<div className="
  grid
  grid-cols-1      // Mobile: 1 սյուն
  sm:grid-cols-2   // Tablet small: 2 սյուն
  md:grid-cols-3   // Tablet: 3 սյուն
  lg:grid-cols-4   // Desktop: 4 սյուն
  gap-4
  md:gap-6
">
  {products.map(product => (
    <ProductCard key={product.id} product={product} />
  ))}
</div>

// Ադապտիվ տիպոգրաֆիկա
<h1 className="
  text-2xl         // Mobile
  md:text-3xl      // Tablet
  lg:text-4xl      // Desktop
  font-bold
">
  Վերնագիր
</h1>
```

---

## ✨ ԱՆԻՄԱՑԻԱՆԵՐ

### Հիմնական անցումներ

```tsx
// ✅ Հարթ անցումներ
<button className="
  transition-colors duration-200
  hover:bg-primary-600
">

// ✅ Էլեմենտների խումբ
<div className="group">
  <img className="
    transition-transform duration-300
    group-hover:scale-105
  " />
</div>
```

### Framer Motion (բարդ անիմացիաների համար)

```tsx
import { motion } from 'framer-motion';

// Խմբի հայտնվելը
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.3 }}
>
  Content
</motion.div>

// Ցանկ stagger-ով
<motion.ul>
  {items.map((item, i) => (
    <motion.li
      key={item.id}
      initial={{ opacity: 0, x: -20 }}
      animate={{ opacity: 1, x: 0 }}
      transition={{ delay: i * 0.1 }}
    >
      {item.name}
    </motion.li>
  ))}
</motion.ul>
```

---

## ♿ ՀԱՍԱՆԵԼԻՈՒԹՅՈՒՆ

### Պարտադիր կանոններ

```tsx
// ✅ Alt նկարների համար
<img src={product.image} alt={product.name} />

// ✅ Label input-ների համար
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// ✅ ARIA ինտերակտիվ էլեմենտների համար
<button aria-label="Փակել մոդալ պատուհանը">
  <XIcon />
</button>

// ✅ Ֆոկուսը տեսանելի
<button className="focus-visible:ring-2 focus-visible:ring-primary-500">

// ✅ Տեքստի կոնտրաստ (WCAG AA)
// Մանր տեքստ: 4.5:1
// Մեծ տեքստ: 3:1
```

---

## 📋 ԴԻԶԱՅՆԻ ՍՏՈՒԳԱՑՈՒՑԱԿ

### Մեկնարկից առաջ

- [ ] Figma-ին մուտք ստացված է
- [ ] Design tokens-ը սահմանված են
- [ ] Tailwind-ը կարգավորված է
- [ ] Հիմնական UI կոմպոնենտները ստեղծված են

### Կոմպոնենտ ստեղծելիս

- [ ] Համապատասխանում է մակետին (pixel perfect)
- [ ] Օգտագործվում են թոքեններ (ոչ կամայական արժեքներ)
- [ ] Բոլոր վիճակները (hover, active, disabled, loading)
- [ ] Responsive բոլոր breakpoints-ում
- [ ] Հասանելիություն (a11y)
- [ ] Անիմացիաները հարթ են

### Հետո

- [ ] Կոմպոնենտը Storybook-ում (ընտրովի)
- [ ] Թեստավորում տարբեր սարքերում
- [ ] Կոնտրաստի ստուգում

---

**Տարբերակ.** 1.0
**Ամսաթիվ.** 2026-02-12

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neetrino)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/neetrino)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
