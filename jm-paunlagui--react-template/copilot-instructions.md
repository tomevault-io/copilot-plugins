## react-template

> > **Persona:** Senior React + Tailwind CSS Designer. Every component is production-grade, accessible, dark-mode-ready, fully dynamic, reusable across projects, and aligned with the Aumovio brand guide.

# CLAUDE.md — Aumovio UI Component Design System

> **Persona:** Senior React + Tailwind CSS Designer. Every component is production-grade, accessible, dark-mode-ready, fully dynamic, reusable across projects, and aligned with the Aumovio brand guide.

---

## Table of Contents

1. [Configuration & Setup](#1-configuration--setup)
2. [Design Tokens & Theming](#2-design-tokens--theming)
3. [Dark Mode](#3-dark-mode)
4. [Icons](#4-icons)
5. [Optimization Guidelines](#5-optimization-guidelines)
6. [Component Library](#6-component-library)
   - [Accordion](#accordion)
   - [Alerts](#alerts)
   - [Avatar](#avatar)
   - [Badge](#badge)
   - [Banner](#banner)
   - [Bottom Navigation](#bottom-navigation)
   - [Breadcrumb](#breadcrumb)
   - [Buttons](#buttons)
   - [Button Group](#button-group)
   - [Card](#card)
   - [Carousel](#carousel)
   - [Chat Bubble](#chat-bubble)
   - [Clipboard](#clipboard)
   - [Datepicker](#datepicker)
   - [Device Mockups](#device-mockups)
   - [Drawer](#drawer)
   - [Dropdowns](#dropdowns)
   - [Footer](#footer)
   - [Gallery](#gallery)
   - [Indicators](#indicators)
   - [Jumbotron](#jumbotron)
   - [KBD](#kbd)
   - [List Group](#list-group)
   - [Mega Menu](#mega-menu)
   - [Modal](#modal)
   - [Navbar](#navbar)
   - [Pagination](#pagination)
   - [Popover](#popover)
   - [Progress](#progress)
   - [Rating](#rating)
   - [Sidebar](#sidebar)
   - [Skeleton](#skeleton)
   - [Speed Dial](#speed-dial)
   - [Spinner](#spinner)
   - [Stepper](#stepper)
   - [Tables](#tables)
   - [Tabs](#tabs)
   - [Timeline](#timeline)
   - [Toast](#toast)
   - [Tooltips](#tooltips)
   - [QR Code](#qr-code)
7. [Forms](#7-forms)
   - [Input Field](#input-field)
   - [File Input](#file-input)
   - [Search Input](#search-input)
   - [Number Input](#number-input)
   - [Phone Input](#phone-input)
   - [Select](#select)
   - [Textarea](#textarea)
   - [Timepicker](#timepicker)
   - [Checkbox](#checkbox)
   - [Radio](#radio)
   - [Toggle](#toggle)
   - [Range](#range)
   - [Floating Label](#floating-label)
8. [Typography](#8-typography)
   - [Headings](#headings)
   - [Paragraphs](#paragraphs)
   - [Blockquote](#blockquote)
   - [Images](#images)
   - [Lists](#lists)
   - [Links](#links)
   - [Text Utilities](#text-utilities)
   - [HR](#hr)
9. [Charts (ApexCharts)](#9-charts-apexcharts)

---

## 1. Configuration & Setup

### Vite + React + Tailwind v4

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
})
```

```css
/* src/assets/styles/index.css */
@import "tailwindcss";
/* All @theme tokens defined here — see Section 2 */
```

### Required Dependencies

```bash
# Core
npm install react react-dom react-router-dom
npm install @vitejs/plugin-react @tailwindcss/vite tailwindcss

# UI Utilities
npm install @headlessui/react @heroicons/react
npm install @fortawesome/react-fontawesome @fortawesome/free-solid-svg-icons @fortawesome/free-regular-svg-icons @fortawesome/free-brands-svg-icons

# Notifications
npm install react-toastify

# Date
npm install date-fns

# Charts
npm install apexcharts react-apexcharts

# Auth / HTTP
npm install axios js-cookie

# QR
npm install qrcode.react
```

### Environment Variables

```env
# .env
VITE_APP_NAME=YourApp
VITE_API_BASE_URL=http://localhost:3000/api/v1/
VITE_LAYOUT_MODE=top        # "top" | "sidebar"
VITE_THEME=light            # "light" | "dark" | "system"
```

---

## 2. Design Tokens & Theming

All tokens live inside `@theme {}` in `src/assets/styles/index.css`.

### Color Hierarchy

| Role      | Token Family       | Hex Anchor | Usage |
|-----------|--------------------|------------|-------|
| Primary   | `orange-*`/`primary-*` | `#FF4208` | 60% — CTAs, active states |
| Secondary | `purple-*`/`secondary-*` | `#4827AF` | 30% — accents, gradients |
| Blue      | `blue-*`           | `#18A9E7`  | Info, links |
| Turquoise | `turquoise-*`      | `#12CAAE`  | Success alt, tags |
| Yellow    | `yellow-*`         | `#CEC43A`  | Highlights, amber |
| Grey      | `grey-*`           | —          | Neutral surfaces |
| Danger    | `danger-*`         | `#D82822`  | Errors, destructive |
| Warn      | `warn-*`           | `#FFD600`  | Warnings |
| Success   | `success-*`        | `#32CB70`  | Confirmations |

### Spacing Scale (Tailwind defaults, referenced as-is)

Use Tailwind's built-in spacing: `p-1`=4px, `p-2`=8px, `p-4`=16px, `p-6`=24px, `p-8`=32px, `p-10`=40px, `p-12`=48px.

### Border Radius Scale

```
rounded-sm   → 2px
rounded      → 4px
rounded-md   → 6px
rounded-lg   → 8px
rounded-xl   → 12px
rounded-2xl  → 16px
rounded-full → 9999px
```

### Shadow Scale

```
shadow-sm   → subtle card lift
shadow      → default cards
shadow-md   → dropdowns, popovers
shadow-lg   → modals, overlays
shadow-xl   → hero elements
shadow-2xl  → focus accent
```

### CSS Custom Properties (for JS access)

```js
// src/utils/tokens.js
export const TOKENS = {
  primary:   '#FF4208',
  secondary: '#4827AF',
  blue:      '#18A9E7',
  turquoise: '#12CAAE',
  yellow:    '#CEC43A',
  danger:    '#D82822',
  warn:      '#FFD600',
  success:   '#32CB70',
  grey: {
    50: '#FAFAFA', 100: '#F0F0F0', 200: '#DCDCDC',
    300: '#C8C8C8', 400: '#AAAAAA', 500: '#787878',
  },
}
```

---

## 3. Dark Mode

### Setup

Dark mode is implemented via a `data-theme` attribute on `<html>` (or a `.dark` class) combined with CSS custom property overrides.

```css
/* index.css — append after @theme */

:root {
  --bg-surface:    #FFFFFF;
  --bg-surface-2:  #FAFAFA;
  --bg-surface-3:  #F0F0F0;
  --text-primary:  #1A1A1A;
  --text-secondary:#484848;
  --text-muted:    #787878;
  --border-color:  #DCDCDC;
}

[data-theme='dark'] {
  --bg-surface:    #0D0D14;
  --bg-surface-2:  #1a1030;
  --bg-surface-3:  #251d3a;
  --text-primary:  #F0F0F0;
  --text-secondary:#C8C8C8;
  --text-muted:    #787878;
  --border-color:  #303030;
}
```

### ThemeContext

```jsx
// src/contexts/theme/ThemeContext.jsx
import { createContext, useCallback, useContext, useEffect, useState } from 'react'

const ThemeContext = createContext(null)
const STORAGE_KEY = 'aumovio-theme'

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(() => {
    const stored = localStorage.getItem(STORAGE_KEY)
    if (stored) return stored
    const pref = import.meta.env.VITE_THEME || 'system'
    if (pref === 'system') return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
    return pref
  })

  useEffect(() => {
    document.documentElement.setAttribute('data-theme', theme)
    localStorage.setItem(STORAGE_KEY, theme)
  }, [theme])

  const toggle = useCallback(() => setTheme(t => t === 'dark' ? 'light' : 'dark'), [])

  return (
    <ThemeContext.Provider value={{ theme, setTheme, toggle, isDark: theme === 'dark' }}>
      {children}
    </ThemeContext.Provider>
  )
}

export function useTheme() {
  const ctx = useContext(ThemeContext)
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider')
  return ctx
}
```

### ThemeToggle Button

```jsx
// src/components/ui/ThemeToggle.jsx
import { SunIcon, MoonIcon } from '@heroicons/react/24/outline'
import { useTheme } from '../../contexts/theme/ThemeContext'

export function ThemeToggle({ size = 'md' }) {
  const { isDark, toggle } = useTheme()
  const sz = size === 'sm' ? 'w-4 h-4' : 'w-5 h-5'
  return (
    <button
      onClick={toggle}
      aria-label="Toggle theme"
      className="p-2 rounded-lg text-grey-500 hover:text-orange-400 hover:bg-orange-400/10
        border border-transparent hover:border-orange-400/20 transition-all duration-200"
    >
      {isDark ? <SunIcon className={sz} /> : <MoonIcon className={sz} />}
    </button>
  )
}
```

### Dark-mode Tailwind Utility Pattern

Since Tailwind v4 uses `data-theme` — add this to `index.css`:

```css
@variant dark (&:where([data-theme=dark], [data-theme=dark] *));
```

Now `dark:` prefix works. Example:

```jsx
<div className="bg-white dark:bg-[#0D0D14] text-black dark:text-white" />
```

---

## 4. Icons

### FontAwesome (Solid, Regular, Brands)

```jsx
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome'
import { faShield, faUser, faCheck } from '@fortawesome/free-solid-svg-icons'
import { faHeart } from '@fortawesome/free-regular-svg-icons'
import { faGithub } from '@fortawesome/free-brands-svg-icons'

<FontAwesomeIcon icon={faShield} className="text-orange-400 w-5 h-5" />
```

### Heroicons (Outline / Solid / Mini)

```jsx
import { ShieldCheckIcon } from '@heroicons/react/24/outline'   // stroke
import { ShieldCheckIcon } from '@heroicons/react/24/solid'     // filled
import { ShieldCheckIcon } from '@heroicons/react/20/solid'     // 20px mini

<ShieldCheckIcon className="w-5 h-5 text-orange-400" />
```

### Icon Size Conventions

| Size  | Class        | Use case                     |
|-------|--------------|------------------------------|
| xs    | `w-3 h-3`    | Inline indicators, badges    |
| sm    | `w-4 h-4`    | Button icons, table actions  |
| md    | `w-5 h-5`    | Nav icons, form icons        |
| lg    | `w-6 h-6`    | Feature icons                |
| xl    | `w-8 h-8`    | Section icons                |
| 2xl   | `w-12 h-12`  | Hero/empty-state icons       |

---

## 5. Optimization Guidelines

### Code Splitting

Always lazy-load views. Never lazy-load shared UI components.

```jsx
// ✅ Correct — lazy views
const InventoryView = lazy(() => import('./features/inventory/Inventory.view'))

// ❌ Wrong — never lazy-load base components
const Button = lazy(() => import('./components/ui/Button'))
```

### Memoization Rules

```jsx
// Memo for expensive pure components
export const DataTable = memo(({ rows, columns }) => { ... })

// useCallback for handlers passed as props
const handleDelete = useCallback((id) => api.delete(id), [])

// useMemo for derived data
const filtered = useMemo(() => rows.filter(r => r.active), [rows])
```

### Image Optimization

```jsx
// Always specify width/height, use loading="lazy" for below-fold
<img
  src={url}
  alt={description}
  width={400}
  height={300}
  loading="lazy"
  decoding="async"
  className="object-cover w-full h-full"
/>
```

### Bundle Size Tips

- Tree-shake FA icons: import individually, never `import * from '@fortawesome/...'`
- ApexCharts: import only the chart types used
- date-fns: import only the functions used

```js
// ✅ Tree-shakeable
import { format, parseISO } from 'date-fns'
import { faUser } from '@fortawesome/free-solid-svg-icons'

// ❌ Entire library
import * as dateFns from 'date-fns'
```

### Accessibility Baseline

Every interactive element must have:
- `aria-label` or visible text
- `:focus-visible` ring using `focus-visible:ring-2 focus-visible:ring-orange-400`
- Keyboard nav support
- `role` where semantics are ambiguous

---

## 6. Component Library

---

### Accordion

```jsx
// src/components/ui/Accordion.jsx
/**
 * Accordion — Collapsible content panels.
 *
 * Props:
 *   items        — [{ id, title, content, icon? }]
 *   defaultOpen  — id of panel open by default (null = all closed)
 *   multiple     — boolean: allow multiple open at once
 *   variant      — 'default' | 'flush' | 'separated'
 *   size         — 'sm' | 'md' | 'lg'
 */
import { useState, useCallback } from 'react'
import { ChevronDownIcon } from '@heroicons/react/24/outline'

const VARIANTS = {
  default:   'border border-grey-200 rounded-xl overflow-hidden divide-y divide-grey-200',
  flush:     'divide-y divide-grey-200',
  separated: 'space-y-2',
}

const ITEM_VARIANTS = {
  default:   '',
  flush:     '',
  separated: 'border border-grey-200 rounded-xl overflow-hidden',
}

const SIZES = {
  sm: { header: 'px-4 py-3 text-sm', body: 'px-4 pb-3 text-sm' },
  md: { header: 'px-5 py-4 text-sm', body: 'px-5 pb-4 text-sm' },
  lg: { header: 'px-6 py-5 text-base', body: 'px-6 pb-5 text-sm' },
}

export function Accordion({
  items = [],
  defaultOpen = null,
  multiple = false,
  variant = 'default',
  size = 'md',
}) {
  const [openIds, setOpenIds] = useState(() =>
    defaultOpen ? [defaultOpen] : []
  )

  const toggle = useCallback((id) => {
    setOpenIds(prev => {
      const isOpen = prev.includes(id)
      if (multiple) return isOpen ? prev.filter(x => x !== id) : [...prev, id]
      return isOpen ? [] : [id]
    })
  }, [multiple])

  const sz = SIZES[size] ?? SIZES.md

  return (
    <div className={`font-aumovio ${VARIANTS[variant] ?? VARIANTS.default}`}>
      {items.map((item) => {
        const isOpen = openIds.includes(item.id)
        return (
          <div key={item.id} className={`bg-white dark:bg-[#1a1030] ${ITEM_VARIANTS[variant]}`}>
            <button
              onClick={() => toggle(item.id)}
              aria-expanded={isOpen}
              className={`w-full flex items-center justify-between gap-3
                text-left font-aumovio-bold text-black/85 dark:text-white/90
                hover:bg-orange-50 dark:hover:bg-orange-400/5
                focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-orange-400/50
                transition-colors duration-200 ${sz.header}`}
            >
              <span className="flex items-center gap-2">
                {item.icon && <item.icon className="w-4 h-4 text-orange-400 shrink-0" />}
                {item.title}
              </span>
              <ChevronDownIcon
                className={`w-4 h-4 text-grey-400 shrink-0 transition-transform duration-300
                  ${isOpen ? 'rotate-180 text-orange-400' : ''}`}
              />
            </button>
            <div
              className={`overflow-hidden transition-all duration-300 ease-in-out
                ${isOpen ? 'max-h-[2000px] opacity-100' : 'max-h-0 opacity-0'}`}
            >
              <div className={`text-black/70 dark:text-white/60 leading-relaxed ${sz.body}`}>
                {item.content}
              </div>
            </div>
          </div>
        )
      })}
    </div>
  )
}

export default Accordion
```

**Usage:**
```jsx
<Accordion
  variant="separated"
  multiple
  items={[
    { id: '1', title: 'What is Aumovio?', content: 'A design system...' },
    { id: '2', title: 'How do I install?', content: 'Run npm install...' },
  ]}
/>
```

---

### Alerts

```jsx
// src/components/ui/Alert.jsx
/**
 * Alert — Feedback message with optional dismiss and actions.
 *
 * Props:
 *   variant   — 'info' | 'success' | 'warning' | 'danger'
 *   title     — string (optional bold heading)
 *   children  — message content
 *   icon      — custom icon component (defaults to variant icon)
 *   dismissible — boolean
 *   onDismiss — () => void
 *   actions   — [{ label, onClick }]
 *   bordered  — boolean
 *   size      — 'sm' | 'md'
 */
import { useState } from 'react'
import { XMarkIcon, InformationCircleIcon, CheckCircleIcon,
  ExclamationTriangleIcon, XCircleIcon } from '@heroicons/react/24/outline'

const CONFIG = {
  info:    { bg: 'bg-purple-100/20 dark:bg-purple-400/10', border: 'border-purple-400/30', text: 'text-purple-500 dark:text-purple-300', icon: InformationCircleIcon },
  success: { bg: 'bg-success-100/40 dark:bg-success-400/10', border: 'border-success-400/30', text: 'text-success-500 dark:text-success-300', icon: CheckCircleIcon },
  warning: { bg: 'bg-warn-100/30 dark:bg-warn-400/10', border: 'border-warn-400/30', text: 'text-warn-500 dark:text-yellow-300', icon: ExclamationTriangleIcon },
  danger:  { bg: 'bg-danger-100/40 dark:bg-danger-400/10', border: 'border-danger-400/30', text: 'text-danger-500 dark:text-danger-300', icon: XCircleIcon },
}

export function Alert({
  variant = 'info',
  title,
  children,
  icon: CustomIcon,
  dismissible = false,
  onDismiss,
  actions = [],
  bordered = true,
  size = 'md',
}) {
  const [visible, setVisible] = useState(true)
  const cfg = CONFIG[variant] ?? CONFIG.info
  const Icon = CustomIcon ?? cfg.icon

  const dismiss = () => {
    setVisible(false)
    onDismiss?.()
  }

  if (!visible) return null

  return (
    <div role="alert" className={`
      flex gap-3 rounded-xl font-aumovio animate-fade-in
      ${size === 'sm' ? 'p-3 text-xs' : 'p-4 text-sm'}
      ${cfg.bg} ${bordered ? `border ${cfg.border}` : ''}
    `}>
      <Icon className={`w-5 h-5 shrink-0 mt-0.5 ${cfg.text}`} />
      <div className="flex-1 min-w-0">
        {title && <p className={`font-aumovio-bold mb-0.5 ${cfg.text}`}>{title}</p>}
        <div className={`text-black/75 dark:text-white/70 leading-relaxed`}>{children}</div>
        {actions.length > 0 && (
          <div className="flex gap-2 mt-3">
            {actions.map((a, i) => (
              <button key={i} onClick={a.onClick}
                className={`text-xs font-aumovio-bold ${cfg.text} hover:underline`}>
                {a.label}
              </button>
            ))}
          </div>
        )}
      </div>
      {dismissible && (
        <button onClick={dismiss} aria-label="Dismiss"
          className="shrink-0 text-grey-400 hover:text-grey-600 transition-colors">
          <XMarkIcon className="w-4 h-4" />
        </button>
      )}
    </div>
  )
}

export default Alert
```

---

### Avatar

```jsx
// src/components/ui/Avatar.jsx
/**
 * Avatar — User profile image or initials fallback.
 *
 * Props:
 *   src       — image URL
 *   name      — full name (used for initials fallback)
 *   size      — 'xs'|'sm'|'md'|'lg'|'xl'|'2xl'
 *   shape     — 'circle' | 'rounded'
 *   status    — 'online'|'offline'|'busy'|'away' (indicator dot)
 *   bordered  — boolean
 *   stacked   — boolean (for Avatar.Group)
 */
const SIZES = {
  xs:  { wrap: 'w-6 h-6',   text: 'text-xs',  dot: 'w-1.5 h-1.5' },
  sm:  { wrap: 'w-8 h-8',   text: 'text-xs',  dot: 'w-2 h-2'     },
  md:  { wrap: 'w-10 h-10', text: 'text-sm',  dot: 'w-2.5 h-2.5' },
  lg:  { wrap: 'w-12 h-12', text: 'text-base',dot: 'w-3 h-3'     },
  xl:  { wrap: 'w-16 h-16', text: 'text-lg',  dot: 'w-3.5 h-3.5' },
  '2xl':{ wrap: 'w-20 h-20',text: 'text-xl',  dot: 'w-4 h-4'     },
}

const STATUS_DOT = {
  online:  'bg-success-400 ring-2 ring-white',
  offline: 'bg-grey-400 ring-2 ring-white',
  busy:    'bg-danger-400 ring-2 ring-white',
  away:    'bg-warn-400 ring-2 ring-white',
}

// Deterministic color from name string
const PALETTE = [
  'bg-orange-400 text-white',
  'bg-purple-400 text-white',
  'bg-blue-400 text-white',
  'bg-turquoise-400 text-white',
  'bg-danger-400 text-white',
  'bg-success-500 text-white',
  'bg-yellow-500 text-black',
]

function getInitials(name = '') {
  const parts = name.trim().split(' ').filter(Boolean)
  if (!parts.length) return '?'
  return parts.length === 1
    ? parts[0].slice(0, 2).toUpperCase()
    : (parts[0][0] + parts[parts.length - 1][0]).toUpperCase()
}

function getColor(name = '') {
  const sum = [...name].reduce((acc, c) => acc + c.charCodeAt(0), 0)
  return PALETTE[sum % PALETTE.length]
}

export function Avatar({
  src,
  name = '',
  size = 'md',
  shape = 'circle',
  status,
  bordered = false,
  stacked = false,
}) {
  const sz = SIZES[size] ?? SIZES.md
  const initials = getInitials(name)
  const color = getColor(name)
  const radius = shape === 'circle' ? 'rounded-full' : 'rounded-lg'

  return (
    <div className={`relative inline-flex shrink-0 ${sz.wrap} ${stacked ? '-ml-3 first:ml-0' : ''}`}>
      {src ? (
        <img
          src={src}
          alt={name || 'Avatar'}
          className={`${sz.wrap} ${radius} object-cover
            ${bordered ? 'ring-2 ring-white dark:ring-grey-800' : ''}`}
        />
      ) : (
        <span className={`flex items-center justify-center w-full h-full font-aumovio-bold
          ${radius} ${color} ${sz.text}
          ${bordered ? 'ring-2 ring-white dark:ring-grey-800' : ''}`}>
          {initials}
        </span>
      )}
      {status && (
        <span className={`absolute bottom-0 right-0 ${sz.dot} rounded-full ${STATUS_DOT[status]}`} />
      )}
    </div>
  )
}

/**
 * AvatarGroup — Stacked row of avatars with overflow count.
 * Props: avatars[], max, size, shape
 */
export function AvatarGroup({ avatars = [], max = 4, size = 'md', shape = 'circle' }) {
  const visible = avatars.slice(0, max)
  const overflow = avatars.length - max
  const sz = SIZES[size] ?? SIZES.md

  return (
    <div className="flex items-center">
      {visible.map((a, i) => (
        <Avatar key={i} {...a} size={size} shape={shape} stacked bordered />
      ))}
      {overflow > 0 && (
        <span className={`-ml-3 flex items-center justify-center ${sz.wrap}
          rounded-full bg-grey-200 dark:bg-grey-700 text-grey-600 dark:text-grey-300
          font-aumovio-bold ${sz.text} ring-2 ring-white dark:ring-grey-800`}>
          +{overflow}
        </span>
      )}
    </div>
  )
}

export default Avatar
```

---

### Badge

```jsx
// src/components/ui/Badge.jsx
/**
 * Badge — Status pill / label.
 *
 * Props:
 *   variant  — 'green'|'red'|'warning'|'blue'|'purple'|'cyan'|'amber'|'grey'|'orange'
 *   size     — 'xs'|'sm'|'md'
 *   dot      — boolean (animated pulse dot)
 *   outline  — boolean (border-only style)
 *   removable — boolean (× button)
 *   onRemove — () => void
 *   pill     — boolean (rounded-full instead of rounded-lg)
 *   children
 */
const V = {
  green:  { solid: 'text-success-400 bg-success-100/60 border-success-400/30',  outline: 'text-success-500 border-success-400 bg-transparent', dot: 'bg-success-400' },
  red:    { solid: 'text-danger-400  bg-danger-100      border-danger-400/30',   outline: 'text-danger-500  border-danger-400  bg-transparent', dot: 'bg-danger-400'  },
  warning:{ solid: 'text-warn-600    bg-warn-100/30     border-warn-400/30',     outline: 'text-warn-600    border-warn-400    bg-transparent', dot: 'bg-warn-400'    },
  blue:   { solid: 'text-blue-500    bg-blue-100/30     border-blue-400/30',     outline: 'text-blue-500    border-blue-400    bg-transparent', dot: 'bg-blue-400'    },
  purple: { solid: 'text-purple-400  bg-purple-100/25   border-purple-400/35',   outline: 'text-purple-500  border-purple-400  bg-transparent', dot: 'bg-purple-400'  },
  cyan:   { solid: 'text-turquoise-500 bg-turquoise-100/22 border-turquoise-400/25', outline: 'text-turquoise-500 border-turquoise-400 bg-transparent', dot: 'bg-turquoise-400' },
  amber:  { solid: 'text-yellow-600  bg-yellow-100      border-yellow-400/30',   outline: 'text-yellow-600  border-yellow-400  bg-transparent', dot: 'bg-yellow-400'  },
  grey:   { solid: 'text-grey-500    bg-grey-100        border-grey-400/30',     outline: 'text-grey-500    border-grey-400    bg-transparent', dot: 'bg-grey-400'    },
  orange: { solid: 'text-orange-500  bg-orange-100/20   border-orange-400/30',   outline: 'text-orange-500  border-orange-400  bg-transparent', dot: 'bg-orange-400'  },
}

const SZ = {
  xs: 'px-1.5 py-0.5 text-xs gap-1',
  sm: 'px-2   py-0.5 text-xs gap-1',
  md: 'px-2.5 py-1   text-xs gap-1.5',
}

export function Badge({
  variant = 'grey', size = 'md', dot = false, outline = false,
  removable = false, onRemove, pill = false, children,
}) {
  const cfg = V[variant] ?? V.grey
  const style = outline ? cfg.outline : cfg.solid

  return (
    <span className={`inline-flex items-center font-aumovio-bold tracking-wide
      border shadow-sm ${pill ? 'rounded-full' : 'rounded-lg'}
      ${style} ${SZ[size] ?? SZ.md}`}>
      {dot && (
        <span className={`w-1.5 h-1.5 rounded-full shrink-0 animate-pulse ${cfg.dot}`} />
      )}
      {children}
      {removable && (
        <button onClick={onRemove} aria-label="Remove" className="ml-0.5 hover:opacity-70">
          <svg width="10" height="10" viewBox="0 0 10 10" fill="none">
            <path d="M8 2L2 8M2 2l6 6" stroke="currentColor" strokeWidth="1.5" strokeLinecap="round"/>
          </svg>
        </button>
      )}
    </span>
  )
}

export default Badge
```

---

### Banner

```jsx
// src/components/ui/Banner.jsx
/**
 * Banner — Full-width announcement bar.
 *
 * Props:
 *   variant    — 'info'|'success'|'warning'|'danger'|'promo'
 *   sticky     — boolean (fixed top)
 *   dismissible — boolean
 *   onDismiss  — () => void
 *   actions    — [{ label, href?, onClick? }]
 *   icon       — icon component
 *   children   — message
 */
import { useState } from 'react'
import { XMarkIcon } from '@heroicons/react/24/outline'
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome'
import { faBullhorn } from '@fortawesome/free-solid-svg-icons'

const BANNERS = {
  info:    'bg-purple-400 text-white',
  success: 'bg-success-500 text-white',
  warning: 'bg-warn-400 text-black',
  danger:  'bg-danger-400 text-white',
  promo:   'bg-gradient-to-r from-orange-400 via-[#ff850a] to-purple-400 text-white',
}

export function Banner({
  variant = 'promo',
  sticky = false,
  dismissible = true,
  onDismiss,
  actions = [],
  icon,
  children,
}) {
  const [visible, setVisible] = useState(true)
  if (!visible) return null

  const dismiss = () => { setVisible(false); onDismiss?.() }

  return (
    <div className={`w-full z-50 font-aumovio ${BANNERS[variant]} ${sticky ? 'sticky top-0' : ''}`}>
      <div className="max-w-7xl mx-auto px-4 py-2.5 flex items-center justify-between gap-4 flex-wrap">
        <div className="flex items-center gap-2 text-sm font-aumovio-bold">
          {icon
            ? <icon className="w-4 h-4 shrink-0" />
            : <FontAwesomeIcon icon={faBullhorn} className="w-4 h-4 shrink-0" />}
          <span>{children}</span>
        </div>
        <div className="flex items-center gap-3">
          {actions.map((a, i) => (
            a.href
              ? <a key={i} href={a.href} className="text-xs underline underline-offset-2 hover:no-underline font-aumovio-bold">{a.label}</a>
              : <button key={i} onClick={a.onClick} className="text-xs underline underline-offset-2 hover:no-underline font-aumovio-bold">{a.label}</button>
          ))}
          {dismissible && (
            <button onClick={dismiss} aria-label="Dismiss banner"
              className="opacity-80 hover:opacity-100 transition-opacity">
              <XMarkIcon className="w-4 h-4" />
            </button>
          )}
        </div>
      </div>
    </div>
  )
}

export default Banner
```

---

### Bottom Navigation

```jsx
// src/components/layout/BottomNav.jsx
/**
 * BottomNav — Mobile fixed bottom navigation bar.
 *
 * Props:
 *   items — [{ id, label, icon, href, badge? }]
 *   active — active item id
 *   onChange — (id) => void
 *   variant — 'default' | 'pill' | 'floating'
 */
import { NavLink } from 'react-router-dom'

export function BottomNav({ items = [], variant = 'default' }) {
  const base = {
    default:  'fixed bottom-0 left-0 right-0 z-50 bg-white dark:bg-[#1a1030] border-t border-grey-200 dark:border-grey-800 px-2 py-1 flex justify-around',
    pill:     'fixed bottom-4 left-1/2 -translate-x-1/2 z-50 bg-white dark:bg-[#1a1030] shadow-2xl rounded-full border border-grey-200 dark:border-grey-700 px-4 py-2 flex gap-1',
    floating: 'fixed bottom-6 left-4 right-4 z-50 bg-white/90 dark:bg-[#1a1030]/90 backdrop-blur-md shadow-2xl rounded-2xl border border-grey-200 dark:border-grey-700 px-3 py-2 flex justify-around',
  }

  return (
    <nav className={`font-aumovio ${base[variant]}`} aria-label="Bottom navigation">
      {items.map((item) => (
        <NavLink key={item.id} to={item.href}>
          {({ isActive }) => (
            <div className={`flex flex-col items-center gap-0.5 px-3 py-1 rounded-xl
              transition-all duration-200 cursor-pointer relative
              ${isActive ? 'text-orange-400' : 'text-grey-400 hover:text-grey-600 dark:hover:text-grey-300'}`}>
              <div className="relative">
                <item.icon className="w-5 h-5" />
                {item.badge && (
                  <span className="absolute -top-1 -right-1 w-4 h-4 bg-danger-400 text-white
                    text-[10px] rounded-full flex items-center justify-center font-aumovio-bold">
                    {item.badge}
                  </span>
                )}
              </div>
              <span className="text-xs">{item.label}</span>
              {isActive && (
                <span className="absolute bottom-0 left-1/2 -translate-x-1/2 w-1 h-1
                  bg-orange-400 rounded-full" />
              )}
            </div>
          )}
        </NavLink>
      ))}
    </nav>
  )
}

export default BottomNav
```

---

### Breadcrumb

```jsx
// src/components/ui/Breadcrumb.jsx
/**
 * Breadcrumb — Page hierarchy navigation.
 *
 * Props:
 *   items     — [{ label, href?, icon? }]  (last item = current, no link)
 *   separator — 'slash' | 'chevron' | 'dot'
 *   size      — 'sm' | 'md'
 *   homeIcon  — boolean (show home icon on first item)
 */
import { NavLink } from 'react-router-dom'
import { HomeIcon, ChevronRightIcon } from '@heroicons/react/24/outline'

const SEPARATORS = {
  slash:   <span className="text-grey-400 select-none">/</span>,
  chevron: <ChevronRightIcon className="w-3.5 h-3.5 text-grey-400 shrink-0" />,
  dot:     <span className="w-1 h-1 rounded-full bg-grey-400 shrink-0" />,
}

export function Breadcrumb({ items = [], separator = 'chevron', size = 'md', homeIcon = true }) {
  const textSz = size === 'sm' ? 'text-xs' : 'text-sm'

  return (
    <nav aria-label="Breadcrumb" className={`flex items-center gap-1.5 flex-wrap font-aumovio ${textSz}`}>
      {items.map((item, i) => {
        const isLast = i === items.length - 1
        return (
          <div key={i} className="flex items-center gap-1.5">
            {i > 0 && (SEPARATORS[separator] ?? SEPARATORS.chevron)}
            {isLast ? (
              <span className="text-orange-400 font-aumovio-bold flex items-center gap-1" aria-current="page">
                {item.icon && <item.icon className="w-3.5 h-3.5" />}
                {item.label}
              </span>
            ) : (
              <NavLink to={item.href ?? '#'}
                className="flex items-center gap-1 text-grey-500 hover:text-orange-400
                  transition-colors duration-200">
                {i === 0 && homeIcon
                  ? <HomeIcon className="w-3.5 h-3.5" />
                  : item.icon && <item.icon className="w-3.5 h-3.5" />}
                {item.label}
              </NavLink>
            )}
          </div>
        )
      })}
    </nav>
  )
}

export default Breadcrumb
```

---

### Buttons

```jsx
// src/components/ui/Button.jsx
/**
 * Button — Polymorphic button component.
 *
 * Props:
 *   variant  — 'primary'|'accent'|'danger'|'warning'|'ghost'|'outline'|'link'|'gradient'
 *   size     — 'xs'|'sm'|'md'|'lg'|'xl'
 *   loading  — boolean
 *   disabled — boolean
 *   fullWidth — boolean
 *   leftIcon  — icon component
 *   rightIcon — icon component
 *   rounded  — boolean (pill shape)
 *   onClick, type, className, children
 */
const BASE = [
  'inline-flex items-center justify-center gap-2 font-aumovio-bold tracking-wide',
  'transition-all duration-300 ease-out',
  'hover:-translate-y-0.5 hover:scale-[1.02] active:scale-[0.98]',
  'backface-hidden border',
  'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2',
  'disabled:opacity-60 disabled:cursor-not-allowed disabled:pointer-events-none',
].join(' ')

const VARIANTS = {
  primary:  'text-orange-400 bg-orange-400/10 border-orange-400/25 hover:bg-orange-400 hover:text-white hover:border-transparent hover:shadow-xl hover:shadow-orange-400/30 focus-visible:ring-orange-400',
  accent:   'text-purple-400 bg-purple-400/10 border-purple-400/20 hover:bg-purple-400 hover:text-white hover:border-transparent hover:shadow-xl hover:shadow-purple-400/30 focus-visible:ring-purple-400',
  danger:   'text-danger-400 bg-danger-100    border-danger-400/20  hover:bg-danger-400 hover:text-white hover:border-transparent hover:shadow-xl hover:shadow-danger-400/30 focus-visible:ring-danger-400',
  warning:  'text-warn-600   bg-warn-100/30   border-warn-400/30    hover:bg-warn-400 hover:text-black hover:border-transparent hover:shadow-xl hover:shadow-warn-400/30 focus-visible:ring-warn-400',
  ghost:    'text-grey-600 dark:text-grey-300 bg-transparent border-grey-200 dark:border-grey-700 hover:bg-grey-100 dark:hover:bg-grey-800 hover:text-black dark:hover:text-white focus-visible:ring-grey-400',
  outline:  'text-orange-400 bg-transparent border-orange-400 hover:bg-orange-400 hover:text-white hover:shadow-lg hover:shadow-orange-400/30 focus-visible:ring-orange-400',
  link:     'text-orange-400 bg-transparent border-transparent hover:underline underline-offset-2 hover:translate-y-0 hover:scale-100 focus-visible:ring-orange-400',
  gradient: 'text-white bg-gradient-to-r from-orange-400 via-[#ff850a] to-purple-400 border-transparent hover:shadow-xl hover:shadow-orange-400/40 hover:brightness-110 focus-visible:ring-orange-400',
}

const SIZES = {
  xs: 'px-2.5 py-1    text-xs rounded-md',
  sm: 'px-3   py-1.5  text-xs rounded-lg',
  md: 'px-4   py-2    text-sm rounded-lg',
  lg: 'px-5   py-2.5  text-base rounded-lg',
  xl: 'px-7   py-3.5  text-base rounded-xl',
}

export default function Button({
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false,
  fullWidth = false,
  leftIcon: LeftIcon,
  rightIcon: RightIcon,
  rounded = false,
  onClick,
  type = 'button',
  className = '',
  children,
}) {
  return (
    <button
      type={type}
      onClick={onClick}
      disabled={disabled || loading}
      className={`${BASE} ${VARIANTS[variant] ?? VARIANTS.primary}
        ${SIZES[size] ?? SIZES.md}
        ${fullWidth ? 'w-full' : ''}
        ${rounded ? '!rounded-full' : ''}
        ${className}`}
    >
      {loading
        ? <span className="w-3.5 h-3.5 border-2 border-current border-t-transparent rounded-full animate-spin shrink-0" />
        : LeftIcon && <LeftIcon className="w-4 h-4 shrink-0" />}
      {children}
      {!loading && RightIcon && <RightIcon className="w-4 h-4 shrink-0" />}
    </button>
  )
}
```

---

### Button Group

```jsx
// src/components/ui/ButtonGroup.jsx
/**
 * ButtonGroup — Attached row of buttons.
 *
 * Props:
 *   items     — [{ id, label, icon?, disabled? }]
 *   active    — id of active item
 *   onChange  — (id) => void
 *   variant   — 'primary' | 'ghost'
 *   size      — 'sm' | 'md' | 'lg'
 *   orientation — 'horizontal' | 'vertical'
 */
const SZ = { sm: 'px-3 py-1.5 text-xs', md: 'px-4 py-2 text-sm', lg: 'px-5 py-2.5 text-sm' }

export function ButtonGroup({
  items = [], active, onChange,
  variant = 'primary', size = 'md', orientation = 'horizontal',
}) {
  const isH = orientation === 'horizontal'

  return (
    <div
      role="group"
      className={`inline-flex font-aumovio-bold
        ${isH ? 'flex-row' : 'flex-col'}
        ${isH ? 'divide-x' : 'divide-y'}
        divide-grey-200 dark:divide-grey-700
        border border-grey-200 dark:border-grey-700
        ${isH ? 'rounded-lg overflow-hidden' : 'rounded-lg overflow-hidden'}`}
    >
      {items.map((item) => {
        const isActive = item.id === active
        return (
          <button
            key={item.id}
            onClick={() => !item.disabled && onChange?.(item.id)}
            disabled={item.disabled}
            className={`flex items-center gap-2 tracking-wide transition-all duration-200
              focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-orange-400/50
              disabled:opacity-50 disabled:cursor-not-allowed
              ${SZ[size] ?? SZ.md}
              ${isActive
                ? 'bg-orange-400 text-white'
                : 'bg-white dark:bg-[#1a1030] text-grey-600 dark:text-grey-300 hover:bg-orange-50 dark:hover:bg-orange-400/10 hover:text-orange-400'
              }`}
          >
            {item.icon && <item.icon className="w-4 h-4 shrink-0" />}
            {item.label}
          </button>
        )
      })}
    </div>
  )
}

export default ButtonGroup
```

---

### Card

```jsx
// src/components/ui/Card.jsx
/**
 * Card — Versatile content container.
 *
 * Props:
 *   variant   — 'default'|'elevated'|'outlined'|'filled'|'glass'
 *   padding   — 'none'|'sm'|'md'|'lg'
 *   hover     — boolean (lift on hover)
 *   clickable — boolean
 *   onClick   — handler
 *   header    — ReactNode
 *   footer    — ReactNode
 *   image     — { src, alt, height? }
 *   children
 */
const VARIANTS = {
  default:  'bg-white dark:bg-[#1a1030] border border-grey-200 dark:border-grey-700 shadow-sm',
  elevated: 'bg-white dark:bg-[#1a1030] shadow-xl border border-grey-100 dark:border-grey-800',
  outlined: 'bg-transparent border-2 border-orange-400/30 dark:border-orange-400/20',
  filled:   'bg-orange-400/5 dark:bg-orange-400/10 border border-orange-400/20',
  glass:    'bg-white/60 dark:bg-white/5 backdrop-blur-md border border-white/30 dark:border-white/10 shadow-xl',
}

const PAD = { none: '', sm: 'p-4', md: 'p-5 md:p-6', lg: 'p-6 md:p-8' }

export function Card({
  variant = 'default', padding = 'md', hover = false, clickable = false,
  onClick, header, footer, image, className = '', children,
}) {
  const Tag = clickable ? 'button' : 'div'

  return (
    <Tag
      onClick={onClick}
      className={`rounded-xl overflow-hidden font-aumovio
        transition-all duration-300
        ${VARIANTS[variant] ?? VARIANTS.default}
        ${hover || clickable ? 'hover:-translate-y-1 hover:shadow-xl cursor-pointer' : ''}
        ${clickable ? 'text-left w-full focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-orange-400' : ''}
        ${className}`}
    >
      {image && (
        <div className={`overflow-hidden ${image.height ?? 'h-48'}`}>
          <img src={image.src} alt={image.alt ?? ''} className="w-full h-full object-cover
            transition-transform duration-500 hover:scale-105" />
        </div>
      )}
      {header && (
        <div className="px-5 py-4 border-b border-grey-200 dark:border-grey-700 font-aumovio-bold text-black/85 dark:text-white/90">
          {header}
        </div>
      )}
      <div className={PAD[padding] ?? PAD.md}>{children}</div>
      {footer && (
        <div className="px-5 py-4 border-t border-grey-200 dark:border-grey-700 bg-grey-50/50 dark:bg-white/5">
          {footer}
        </div>
      )}
    </Tag>
  )
}

export default Card
```

---

### Carousel

```jsx
// src/components/ui/Carousel.jsx
/**
 * Carousel — Image / content slider.
 *
 * Props:
 *   items       — [{ id, content: ReactNode }]  OR  [{ id, src, alt, caption? }]
 *   autoPlay    — boolean
 *   interval    — ms (default 4000)
 *   showDots    — boolean
 *   showArrows  — boolean
 *   loop        — boolean
 *   transition  — 'slide' | 'fade'
 */
import { useState, useEffect, useCallback, useRef } from 'react'
import { ChevronLeftIcon, ChevronRightIcon } from '@heroicons/react/24/outline'

export function Carousel({
  items = [], autoPlay = true, interval = 4000,
  showDots = true, showArrows = true, loop = true, transition = 'slide',
}) {
  const [current, setCurrent] = useState(0)
  const timerRef = useRef(null)

  const count = items.length

  const go = useCallback((idx) => {
    let next = idx
    if (loop) next = ((idx % count) + count) % count
    else next = Math.max(0, Math.min(count - 1, idx))
    setCurrent(next)
  }, [count, loop])

  useEffect(() => {
    if (!autoPlay) return
    timerRef.current = setInterval(() => go(current + 1), interval)
    return () => clearInterval(timerRef.current)
  }, [autoPlay, current, go, interval])

  return (
    <div className="relative overflow-hidden rounded-xl select-none">
      {/* Slides */}
      <div className={`flex transition-transform duration-500 ease-in-out`}
        style={{ transform: transition === 'slide' ? `translateX(-${current * 100}%)` : undefined }}>
        {items.map((item, i) => (
          <div key={item.id ?? i} className={`w-full shrink-0
            ${transition === 'fade' ? `absolute inset-0 transition-opacity duration-500 ${i === current ? 'opacity-100 z-10' : 'opacity-0 z-0'}` : ''}`}>
            {item.src
              ? <img src={item.src} alt={item.alt ?? ''} className="w-full h-64 md:h-80 object-cover" />
              : item.content}
            {item.caption && (
              <div className="absolute bottom-0 left-0 right-0 bg-black/50 text-white
                px-4 py-2 text-sm font-aumovio backdrop-blur-sm">
                {item.caption}
              </div>
            )}
          </div>
        ))}
      </div>

      {/* Arrows */}
      {showArrows && (
        <>
          <button onClick={() => go(current - 1)}
            className="absolute left-2 top-1/2 -translate-y-1/2 z-20
              w-8 h-8 rounded-full bg-white/80 dark:bg-black/50 shadow
              flex items-center justify-center hover:bg-white transition-colors">
            <ChevronLeftIcon className="w-4 h-4 text-grey-700" />
          </button>
          <button onClick={() => go(current + 1)}
            className="absolute right-2 top-1/2 -translate-y-1/2 z-20
              w-8 h-8 rounded-full bg-white/80 dark:bg-black/50 shadow
              flex items-center justify-center hover:bg-white transition-colors">
            <ChevronRightIcon className="w-4 h-4 text-grey-700" />
          </button>
        </>
      )}

      {/* Dots */}
      {showDots && (
        <div className="absolute bottom-3 left-1/2 -translate-x-1/2 z-20 flex gap-1.5">
          {items.map((_, i) => (
            <button key={i} onClick={() => go(i)}
              className={`rounded-full transition-all duration-300
                ${i === current ? 'w-5 h-2 bg-orange-400' : 'w-2 h-2 bg-white/60 hover:bg-white'}`}
            />
          ))}
        </div>
      )}
    </div>
  )
}

export default Carousel
```

---

### Chat Bubble

```jsx
// src/components/ui/ChatBubble.jsx
/**
 * ChatBubble — Message in a chat interface.
 *
 * Props:
 *   message   — string
 *   sender    — { name, avatar?, initials? }
 *   timestamp — string
 *   position  — 'left' | 'right'
 *   status    — 'sent'|'delivered'|'read' (right only)
 *   type      — 'text'|'image'|'file'
 *   imageUrl  — string (when type='image')
 *   fileName  — string (when type='file')
 *   reactions — [{ emoji, count }]
 */
import Avatar from './Avatar'

const STATUS_ICONS = {
  sent:      '✓',
  delivered: '✓✓',
  read:      <span className="text-blue-400">✓✓</span>,
}

export function ChatBubble({
  message, sender, timestamp, position = 'left',
  status, type = 'text', imageUrl, fileName, reactions = [],
}) {
  const isRight = position === 'right'

  return (
    <div className={`flex items-end gap-2 font-aumovio ${isRight ? 'flex-row-reverse' : 'flex-row'}`}>
      {!isRight && (
        <Avatar name={sender?.name} src={sender?.avatar} size="sm" />
      )}
      <div className={`max-w-xs md:max-w-sm lg:max-w-md ${isRight ? 'items-end' : 'items-start'} flex flex-col gap-1`}>
        {!isRight && sender?.name && (
          <span className="text-xs text-grey-500 font-aumovio-bold px-1">{sender.name}</span>
        )}
        <div className={`relative rounded-2xl px-4 py-2.5 text-sm leading-relaxed
          ${isRight
            ? 'bg-orange-400 text-white rounded-br-sm'
            : 'bg-grey-100 dark:bg-grey-800 text-black/85 dark:text-white/85 rounded-bl-sm'}`}>
          {type === 'image' && imageUrl && (
            <img src={imageUrl} alt="Shared" className="rounded-lg max-w-full mb-1" />
          )}
          {type === 'file' && (
            <div className="flex items-center gap-2 text-xs">
              <span className="text-2xl">📎</span>
              <span className="underline">{fileName}</span>
            </div>
          )}
          {message && <p>{message}</p>}
        </div>
        {reactions.length > 0 && (
          <div className={`flex gap-1 flex-wrap ${isRight ? 'justify-end' : ''}`}>
            {reactions.map((r, i) => (
              <span key={i} className="text-xs bg-white dark:bg-grey-800 border border-grey-200
                dark:border-grey-700 rounded-full px-1.5 py-0.5 shadow-sm">
                {r.emoji} {r.count}
              </span>
            ))}
          </div>
        )}
        <div className={`flex items-center gap-1 text-xs text-grey-400 ${isRight ? 'flex-row-reverse' : ''}`}>
          <span>{timestamp}</span>
          {isRight && status && <span>{STATUS_ICONS[status] ?? STATUS_ICONS.sent}</span>}
        </div>
      </div>
    </div>
  )
}

export default ChatBubble
```

---

### Clipboard

```jsx
// src/components/ui/Clipboard.jsx
/**
 * Clipboard — Copy-to-clipboard with visual feedback.
 *
 * Props:
 *   value    — string to copy
 *   label    — display text (defaults to value)
 *   showCode — boolean (shows as code block)
 *   variant  — 'inline' | 'block'
 */
import { useState, useCallback } from 'react'
import { ClipboardIcon, ClipboardDocumentCheckIcon } from '@heroicons/react/24/outline'

export function Clipboard({ value = '', label, showCode = false, variant = 'inline' }) {
  const [copied, setCopied] = useState(false)

  const copy = useCallback(async () => {
    try {
      await navigator.clipboard.writeText(value)
      setCopied(true)
      setTimeout(() => setCopied(false), 2000)
    } catch { /* fallback: execCommand */ }
  }, [value])

  if (variant === 'block') {
    return (
      <div className="relative rounded-xl bg-grey-900 dark:bg-black border border-grey-700 font-mono">
        <div className="flex items-center justify-between px-4 py-2 border-b border-grey-700">
          <span className="text-xs text-grey-400">{label ?? 'Code'}</span>
          <button onClick={copy}
            className={`flex items-center gap-1.5 text-xs transition-colors
              ${copied ? 'text-success-400' : 'text-grey-400 hover:text-white'}`}>
            {copied
              ? <><ClipboardDocumentCheckIcon className="w-4 h-4" /> Copied!</>
              : <><ClipboardIcon className="w-4 h-4" /> Copy</>}
          </button>
        </div>
        <pre className="px-4 py-4 text-sm text-grey-100 overflow-x-auto">
          <code>{value}</code>
        </pre>
      </div>
    )
  }

  return (
    <div className="inline-flex items-center gap-2 bg-grey-100 dark:bg-grey-800
      border border-grey-200 dark:border-grey-700 rounded-lg px-3 py-2 font-aumovio">
      <span className="text-sm text-black/70 dark:text-white/70 font-mono truncate max-w-xs">
        {label ?? value}
      </span>
      <button onClick={copy}
        className={`transition-all duration-200 shrink-0
          ${copied ? 'text-success-400 scale-110' : 'text-grey-400 hover:text-orange-400'}`}
        aria-label="Copy to clipboard">
        {copied
          ? <ClipboardDocumentCheckIcon className="w-4 h-4" />
          : <ClipboardIcon className="w-4 h-4" />}
      </button>
    </div>
  )
}

export default Clipboard
```

---

### Datepicker

```jsx
// src/components/ui/Datepicker.jsx
/**
 * Datepicker — Calendar date selection.
 *
 * Props:
 *   value      — Date | null
 *   onChange   — (date: Date) => void
 *   placeholder — string
 *   minDate    — Date
 *   maxDate    — Date
 *   disabled   — boolean
 *   range      — boolean (returns [startDate, endDate])
 */
import { useState, useRef, useEffect } from 'react'
import { format, startOfMonth, endOfMonth, eachDayOfInterval,
  startOfWeek, endOfWeek, isSameMonth, isSameDay, isToday,
  addMonths, subMonths, isBefore, isAfter } from 'date-fns'
import { CalendarIcon, ChevronLeftIcon, ChevronRightIcon } from '@heroicons/react/24/outline'

const DAYS = ['Su','Mo','Tu','We','Th','Fr','Sa']

export function Datepicker({
  value = null, onChange, placeholder = 'Select date',
  minDate, maxDate, disabled = false,
}) {
  const [open, setOpen] = useState(false)
  const [viewDate, setViewDate] = useState(value ?? new Date())
  const ref = useRef(null)

  useEffect(() => {
    const handler = (e) => { if (!ref.current?.contains(e.target)) setOpen(false) }
    document.addEventListener('mousedown', handler)
    return () => document.removeEventListener('mousedown', handler)
  }, [])

  const days = eachDayOfInterval({
    start: startOfWeek(startOfMonth(viewDate)),
    end:   endOfWeek(endOfMonth(viewDate)),
  })

  const isDisabled = (d) => (minDate && isBefore(d, minDate)) || (maxDate && isAfter(d, maxDate))

  return (
    <div ref={ref} className="relative inline-block font-aumovio w-full">
      <button
        type="button"
        onClick={() => !disabled && setOpen(o => !o)}
        disabled={disabled}
        className={`w-full flex items-center gap-2 px-3 py-2 rounded-lg border text-sm text-left
          bg-white dark:bg-[#1a1030] text-black/80 dark:text-white/80
          transition-all duration-200
          ${open ? 'border-orange-400 ring-2 ring-orange-400/30 shadow-md' : 'border-grey-300 dark:border-grey-700 hover:border-orange-400/50'}
          ${disabled ? 'opacity-50 cursor-not-allowed' : ''}`}>
        <CalendarIcon className="w-4 h-4 text-grey-400 shrink-0" />
        <span className={value ? 'text-black dark:text-white' : 'text-grey-400'}>
          {value ? format(value, 'MMM dd, yyyy') : placeholder}
        </span>
      </button>

      {open && (
        <div className="absolute top-full left-0 mt-2 z-50 bg-white dark:bg-[#1a1030]
          border border-grey-200 dark:border-grey-700 rounded-xl shadow-2xl p-4 w-72 animate-scale-in">
          {/* Header */}
          <div className="flex items-center justify-between mb-4">
            <button onClick={() => setViewDate(subMonths(viewDate, 1))}
              className="p-1.5 rounded-lg hover:bg-orange-50 dark:hover:bg-orange-400/10 text-grey-500 hover:text-orange-400 transition-colors">
              <ChevronLeftIcon className="w-4 h-4" />
            </button>
            <span className="font-aumovio-bold text-sm text-black/85 dark:text-white/90">
              {format(viewDate, 'MMMM yyyy')}
            </span>
            <button onClick={() => setViewDate(addMonths(viewDate, 1))}
              className="p-1.5 rounded-lg hover:bg-orange-50 dark:hover:bg-orange-400/10 text-grey-500 hover:text-orange-400 transition-colors">
              <ChevronRightIcon className="w-4 h-4" />
            </button>
          </div>
          {/* Day names */}
          <div className="grid grid-cols-7 mb-2">
            {DAYS.map(d => (
              <span key={d} className="text-center text-xs font-aumovio-bold text-grey-400 py-1">{d}</span>
            ))}
          </div>
          {/* Days */}
          <div className="grid grid-cols-7 gap-0.5">
            {days.map((day, i) => {
              const outside = !isSameMonth(day, viewDate)
              const selected = value && isSameDay(day, value)
              const today = isToday(day)
              const dis = isDisabled(day)
              return (
                <button
                  key={i}
                  onClick={() => { if (!dis) { onChange?.(day); setOpen(false) } }}
                  disabled={dis}
                  className={`h-8 w-8 mx-auto rounded-lg text-xs transition-all duration-150
                    ${selected ? 'bg-orange-400 text-white font-aumovio-bold shadow-md' : ''}
                    ${today && !selected ? 'ring-2 ring-orange-400 text-orange-400 font-aumovio-bold' : ''}
                    ${outside ? 'text-grey-300 dark:text-grey-600' : 'text-black/80 dark:text-white/80'}
                    ${dis ? 'opacity-30 cursor-not-allowed' : 'hover:bg-orange-50 dark:hover:bg-orange-400/10 hover:text-orange-400'}`}
                >
                  {format(day, 'd')}
                </button>
              )
            })}
          </div>
          {/* Today shortcut */}
          <div className="mt-3 pt-3 border-t border-grey-100 dark:border-grey-700">
            <button onClick={() => { onChange?.(new Date()); setOpen(false) }}
              className="text-xs text-orange-400 hover:underline font-aumovio-bold">
              Today
            </button>
          </div>
        </div>
      )}
    </div>
  )
}

export default Datepicker
```

---

### Device Mockups

```jsx
// src/components/ui/DeviceMockup.jsx
/**
 * DeviceMockup — Phone/Tablet/Browser frame for screenshots.
 *
 * Props:
 *   device   — 'phone'|'tablet'|'browser'|'desktop'
 *   color    — 'dark'|'light'|'orange'
 *   children — content inside the screen
 */
export function DeviceMockup({ device = 'phone', color = 'dark', children }) {
  if (device === 'browser') return (
    <div className={`rounded-xl overflow-hidden shadow-2xl border font-aumovio
      ${color === 'dark' ? 'bg-grey-900 border-grey-700' : 'bg-grey-100 border-grey-300'}`}>
      <div className={`flex items-center gap-2 px-4 py-3 border-b
        ${color === 'dark' ? 'border-grey-700 bg-grey-800' : 'border-grey-200 bg-grey-50'}`}>
        <div className="flex gap-1.5">
          {['bg-danger-400','bg-warn-400','bg-success-400'].map((c, i) => (
            <span key={i} className={`w-3 h-3 rounded-full ${c}`} />
          ))}
        </div>
        <div className={`flex-1 mx-4 rounded-md px-3 py-1 text-xs truncate
          ${color === 'dark' ? 'bg-grey-700 text-grey-300' : 'bg-white text-grey-500 border border-grey-200'}`}>
          https://yourapp.com
        </div>
      </div>
      <div className="overflow-hidden">{children}</div>
    </div>
  )

  if (device === 'phone') return (
    <div className={`relative mx-auto w-64 rounded-[3rem] shadow-2xl border-4 overflow-hidden
      ${color === 'dark' ? 'bg-grey-900 border-grey-700' : 'bg-white border-grey-300'}`}
      style={{ paddingTop: '2.5rem', paddingBottom: '2rem' }}>
      {/* Notch */}
      <div className={`absolute top-3 left-1/2 -translate-x-1/2 w-20 h-5 rounded-full
        ${color === 'dark' ? 'bg-grey-800' : 'bg-grey-200'}`} />
      {/* Home indicator */}
      <div className={`absolute bottom-2 left-1/2 -translate-x-1/2 w-24 h-1 rounded-full
        ${color === 'dark' ? 'bg-grey-600' : 'bg-grey-300'}`} />
      <div className="overflow-y-auto max-h-[500px]">{children}</div>
    </div>
  )

  return <div className="rounded-xl overflow-hidden shadow-2xl">{children}</div>
}

export default DeviceMockup
```

---

### Drawer

```jsx
// src/components/ui/Drawer.jsx
/**
 * Drawer — Slide-in panel from any edge.
 *
 * Props:
 *   open      — boolean
 *   onClose   — () => void
 *   side      — 'left'|'right'|'top'|'bottom'
 *   size      — 'sm'|'md'|'lg'|'xl'|'full'
 *   title     — string
 *   backdrop  — boolean
 *   children
 */
import { useEffect } from 'react'
import { XMarkIcon } from '@heroicons/react/24/outline'

const TRANSLATE = {
  left:   { closed: '-translate-x-full', open: 'translate-x-0', pos: 'left-0 top-0 bottom-0' },
  right:  { closed: 'translate-x-full',  open: 'translate-x-0', pos: 'right-0 top-0 bottom-0' },
  top:    { closed: '-translate-y-full', open: 'translate-y-0', pos: 'top-0 left-0 right-0' },
  bottom: { closed: 'translate-y-full',  open: 'translate-y-0', pos: 'bottom-0 left-0 right-0' },
}

const WIDTHS = {
  sm: 'w-64', md: 'w-80', lg: 'w-96', xl: 'w-[480px]', full: 'w-full'
}

const HEIGHTS = {
  sm: 'h-64', md: 'h-80', lg: 'h-96', xl: 'h-[480px]', full: 'h-full'
}

export function Drawer({ open, onClose, side = 'right', size = 'md', title, backdrop = true, children }) {
  useEffect(() => {
    if (!open) return
    const h = (e) => { if (e.key === 'Escape') onClose?.() }
    document.addEventListener('keydown', h)
    document.body.style.overflow = 'hidden'
    return () => { document.removeEventListener('keydown', h); document.body.style.overflow = '' }
  }, [open, onClose])

  const t = TRANSLATE[side] ?? TRANSLATE.right
  const isH = side === 'left' || side === 'right'
  const dim = isH ? WIDTHS[size] : HEIGHTS[size]

  return (
    <>
      {backdrop && (
        <div onClick={onClose}
          className={`fixed inset-0 z-40 bg-black/40 backdrop-blur-sm transition-opacity duration-300
            ${open ? 'opacity-100 pointer-events-auto' : 'opacity-0 pointer-events-none'}`} />
      )}
      <div
        role="dialog" aria-modal="true"
        className={`fixed z-50 ${t.pos} ${dim} ${isH ? 'h-full' : 'w-full'}
          bg-white dark:bg-[#1a1030] shadow-2xl
          transform transition-transform duration-300 ease-in-out font-aumovio
          flex flex-col
          ${open ? t.open : t.closed}`}>
        {/* Header */}
        <div className="flex items-center justify-between px-5 py-4 border-b border-grey-200 dark:border-grey-700 shrink-0">
          <h2 className="font-aumovio-bold text-base text-black/85 dark:text-white/90">
            {title}
          </h2>
          <button onClick={onClose} aria-label="Close drawer"
            className="p-1.5 rounded-lg text-grey-400 hover:text-grey-600 hover:bg-grey-100
              dark:hover:bg-grey-800 transition-colors">
            <XMarkIcon className="w-5 h-5" />
          </button>
        </div>
        {/* Body */}
        <div className="flex-1 overflow-y-auto p-5">{children}</div>
      </div>
    </>
  )
}

export default Drawer
```

---

### Dropdowns

```jsx
// src/components/ui/Dropdown.jsx
/**
 * Dropdown — Contextual menu on click/hover.
 *
 * Props:
 *   trigger  — ReactNode (the button/element that opens it)
 *   items    — [{ id, label, icon?, shortcut?, divider?, danger?, disabled?, onClick }]
 *   placement — 'bottom-start'|'bottom-end'|'top-start'|'top-end'
 *   width    — 'auto'|'sm'|'md'|'lg' (sm=160,md=192,lg=224)
 */
import { useState, useRef, useEffect } from 'react'

const WIDTHS = { auto: 'min-w-max', sm: 'w-40', md: 'w-48', lg: 'w-56' }
const PLACEMENT = {
  'bottom-start': 'top-full left-0 mt-1',
  'bottom-end':   'top-full right-0 mt-1',
  'top-start':    'bottom-full left-0 mb-1',
  'top-end':      'bottom-full right-0 mb-1',
}

export function Dropdown({
  trigger, items = [], placement = 'bottom-start', width = 'md'
}) {
  const [open, setOpen] = useState(false)
  const ref = useRef(null)

  useEffect(() => {
    const h = (e) => { if (!ref.current?.contains(e.target)) setOpen(false) }
    document.addEventListener('mousedown', h)
    return () => document.removeEventListener('mousedown', h)
  }, [])

  return (
    <div ref={ref} className="relative inline-block font-aumovio">
      <div onClick={() => setOpen(o => !o)} className="cursor-pointer">{trigger}</div>

      {open && (
        <div className={`absolute z-50 ${PLACEMENT[placement]} ${WIDTHS[width]}
          bg-white dark:bg-[#1a1030] border border-grey-200 dark:border-grey-700
          rounded-xl shadow-2xl py-1.5 animate-scale-in origin-top`}>
          {items.map((item, i) => {
            if (item.divider) return (
              <div key={i} className="my-1.5 border-t border-grey-200 dark:border-grey-700" />
            )
            return (
              <button key={item.id ?? i}
                onClick={() => { if (!item.disabled) { item.onClick?.(); setOpen(false) } }}
                disabled={item.disabled}
                className={`w-full flex items-center gap-2.5 px-3.5 py-2 text-sm text-left
                  transition-colors duration-150 font-aumovio
                  ${item.disabled ? 'opacity-40 cursor-not-allowed' : 'cursor-pointer'}
                  ${item.danger
                    ? 'text-danger-500 hover:bg-danger-100 dark:hover:bg-danger-400/10'
                    : 'text-black/80 dark:text-white/80 hover:bg-orange-50 dark:hover:bg-orange-400/5 hover:text-orange-500'
                  }`}>
                {item.icon && <item.icon className="w-4 h-4 shrink-0" />}
                <span className="flex-1">{item.label}</span>
                {item.shortcut && (
                  <kbd className="text-xs text-grey-400 font-mono bg-grey-100 dark:bg-grey-800
                    border border-grey-200 dark:border-grey-700 rounded px-1.5 py-0.5">
                    {item.shortcut}
                  </kbd>
                )}
              </button>
            )
          })}
        </div>
      )}
    </div>
  )
}

export default Dropdown
```

---

### Gallery

```jsx
// src/components/ui/Gallery.jsx
/**
 * Gallery — Responsive image grid with lightbox.
 *
 * Props:
 *   images     — [{ src, alt, caption? }]
 *   columns    — 2|3|4|5
 *   gap        — 'sm'|'md'|'lg'
 *   lightbox   — boolean
 *   masonry    — boolean
 */
import { useState } from 'react'
import { XMarkIcon, ChevronLeftIcon, ChevronRightIcon } from '@heroicons/react/24/outline'

export function Gallery({ images = [], columns = 3, gap = 'md', lightbox = true }) {
  const [selected, setSelected] = useState(null)
  const COLS = { 2:'grid-cols-2',3:'grid-cols-2 md:grid-cols-3',4:'grid-cols-2 md:grid-cols-4',5:'grid-cols-2 md:grid-cols-5' }
  const GAPS = { sm:'gap-1',md:'gap-3',lg:'gap-6' }

  return (
    <>
      <div className={`grid ${COLS[columns]??COLS[3]} ${GAPS[gap]??GAPS.md}`}>
        {images.map((img, i) => (
          <div key={i} onClick={() => lightbox && setSelected(i)}
            className={`overflow-hidden rounded-xl bg-grey-100 dark:bg-grey-800
              ${lightbox ? 'cursor-zoom-in' : ''} group`}>
            <img src={img.src} alt={img.alt ?? ''}
              className="w-full h-48 object-cover transition-transform duration-500 group-hover:scale-110" />
          </div>
        ))}
      </div>

      {/* Lightbox */}
      {lightbox && selected !== null && (
        <div className="fixed inset-0 z-50 bg-black/90 flex items-center justify-center p-4"
          onClick={() => setSelected(null)}>
          <button onClick={() => setSelected(null)}
            className="absolute top-4 right-4 text-white/70 hover:text-white">
            <XMarkIcon className="w-8 h-8" />
          </button>
          {selected > 0 && (
            <button onClick={(e) => { e.stopPropagation(); setSelected(s => s - 1) }}
              className="absolute left-4 top-1/2 -translate-y-1/2 text-white/70 hover:text-white">
              <ChevronLeftIcon className="w-8 h-8" />
            </button>
          )}
          <img src={images[selected]?.src} alt={images[selected]?.alt}
            className="max-h-[90vh] max-w-full rounded-xl shadow-2xl"
            onClick={(e) => e.stopPropagation()} />
          {selected < images.length - 1 && (
            <button onClick={(e) => { e.stopPropagation(); setSelected(s => s + 1) }}
              className="absolute right-4 top-1/2 -translate-y-1/2 text-white/70 hover:text-white">
              <ChevronRightIcon className="w-8 h-8" />
            </button>
          )}
          {images[selected]?.caption && (
            <p className="absolute bottom-6 left-1/2 -translate-x-1/2 text-white/80 text-sm">
              {images[selected].caption}
            </p>
          )}
        </div>
      )}
    </>
  )
}

export default Gallery
```

---

### Indicators

```jsx
// src/components/ui/Indicator.jsx
/**
 * Indicator — Overlay badge on a UI element (notification dot, count, status).
 *
 * Props:
 *   children  — the element to wrap
 *   content   — string | number (shown in badge; omit for dot-only)
 *   color     — 'orange'|'danger'|'success'|'warn'|'purple'|'blue'|'grey'
 *   position  — 'top-right'|'top-left'|'bottom-right'|'bottom-left'
 *   pulse     — boolean
 *   max       — number (99+ truncation)
 *   hidden    — boolean
 */
const COLORS = {
  orange:  'bg-orange-400 text-white',
  danger:  'bg-danger-400 text-white',
  success: 'bg-success-400 text-white',
  warn:    'bg-warn-400 text-black',
  purple:  'bg-purple-400 text-white',
  blue:    'bg-blue-400 text-white',
  grey:    'bg-grey-400 text-white',
}

const POS = {
  'top-right':    '-top-1 -right-1',
  'top-left':     '-top-1 -left-1',
  'bottom-right': '-bottom-1 -right-1',
  'bottom-left':  '-bottom-1 -left-1',
}

export function Indicator({
  children, content, color = 'danger', position = 'top-right',
  pulse = false, max = 99, hidden = false,
}) {
  const display = typeof content === 'number' && content > max ? `${max}+` : content
  const hasContent = display !== undefined && display !== null && display !== ''

  return (
    <div className="relative inline-flex">
      {children}
      {!hidden && (
        <span className={`absolute ${POS[position]} z-10 flex items-center justify-center
          font-aumovio-bold tracking-wide
          ${hasContent ? 'min-w-[1.25rem] h-5 px-1 text-[10px] rounded-full' : 'w-2.5 h-2.5 rounded-full'}
          ${COLORS[color] ?? COLORS.danger}
          ${pulse ? 'animate-pulse' : ''}`}>
          {hasContent ? display : null}
        </span>
      )}
    </div>
  )
}

export default Indicator
```

---

### Jumbotron

```jsx
// src/components/ui/Jumbotron.jsx
/**
 * Jumbotron — Full-width hero/CTA section.
 *
 * Props:
 *   title, subtitle, description
 *   primaryAction  — { label, onClick, href? }
 *   secondaryAction — { label, onClick, href? }
 *   backgroundImage — string
 *   gradient       — boolean
 *   align          — 'left'|'center'|'right'
 *   size           — 'sm'|'md'|'lg'|'full'
 */
const ALIGNS = { left: 'text-left items-start', center: 'text-center items-center', right: 'text-right items-end' }
const SIZES  = { sm:'py-12',md:'py-20',lg:'py-32',full:'min-h-screen flex items-center' }

export function Jumbotron({
  title, subtitle, description,
  primaryAction, secondaryAction,
  backgroundImage, gradient = true,
  align = 'center', size = 'lg',
}) {
  return (
    <section
      className={`relative w-full overflow-hidden font-aumovio ${SIZES[size]}`}
      style={backgroundImage ? { backgroundImage: `url(${backgroundImage})`, backgroundSize:'cover', backgroundPosition:'center' } : {}}
    >
      {/* Overlay */}
      {(backgroundImage || gradient) && (
        <div className={`absolute inset-0 ${gradient && !backgroundImage
          ? 'bg-gradient-to-br from-[#ff850a] via-orange-400 to-purple-400'
          : 'bg-black/50 backdrop-blur-sm'}`} />
      )}

      <div className={`relative z-10 max-w-5xl mx-auto px-6
        flex flex-col gap-6 ${ALIGNS[align] ?? ALIGNS.center}`}>
        {subtitle && (
          <span className={`inline-flex items-center px-3 py-1 rounded-full text-xs font-aumovio-bold
            uppercase tracking-widest
            ${backgroundImage || gradient
              ? 'bg-white/20 text-white border border-white/30'
              : 'bg-orange-400/10 text-orange-400 border border-orange-400/20'}`}>
            {subtitle}
          </span>
        )}
        <h1 className={`text-4xl md:text-5xl lg:text-6xl font-extrabold leading-tight tracking-tight
          ${backgroundImage || gradient ? 'text-white drop-shadow-2xl' : 'text-black dark:text-white'}`}>
          {title}
        </h1>
        {description && (
          <p className={`max-w-2xl text-lg leading-relaxed
            ${backgroundImage || gradient ? 'text-white/80' : 'text-black/60 dark:text-white/60'}`}>
            {description}
          </p>
        )}
        {(primaryAction || secondaryAction) && (
          <div className={`flex gap-4 flex-wrap ${align === 'center' ? 'justify-center' : ''}`}>
            {primaryAction && (
              <a href={primaryAction.href ?? '#'} onClick={primaryAction.onClick}
                className="px-6 py-3 rounded-lg font-aumovio-bold text-sm
                  bg-white text-orange-400 hover:bg-orange-50
                  shadow-lg hover:shadow-xl transition-all duration-300 hover:-translate-y-0.5">
                {primaryAction.label}
              </a>
            )}
            {secondaryAction && (
              <a href={secondaryAction.href ?? '#'} onClick={secondaryAction.onClick}
                className="px-6 py-3 rounded-lg font-aumovio-bold text-sm
                  border-2 border-white/60 text-white hover:bg-white/10
                  transition-all duration-300 hover:-translate-y-0.5">
                {secondaryAction.label}
              </a>
            )}
          </div>
        )}
      </div>
    </section>
  )
}

export default Jumbotron
```

---

### KBD

```jsx
// src/components/ui/KBD.jsx
/**
 * KBD — Keyboard shortcut display.
 *
 * Props:
 *   keys    — string[] (each key rendered as its own pill)
 *   size    — 'sm'|'md'|'lg'
 *   variant — 'default'|'dark'
 */
const SZ = { sm:'text-[10px] px-1.5 py-0.5', md:'text-xs px-2 py-1', lg:'text-sm px-2.5 py-1.5' }
const V  = {
  default: 'bg-white dark:bg-grey-800 border border-grey-300 dark:border-grey-600 text-grey-700 dark:text-grey-200 shadow-sm',
  dark:    'bg-grey-800 dark:bg-grey-900 border border-grey-700 text-grey-200 shadow-sm',
}

export function KBD({ keys = [], size = 'md', variant = 'default' }) {
  return (
    <span className="inline-flex items-center gap-1 font-mono">
      {keys.map((key, i) => (
        <>
          {i > 0 && <span key={`plus-${i}`} className="text-grey-400 text-xs">+</span>}
          <kbd key={key} className={`inline-flex items-center justify-center rounded font-aumovio-bold
            ${SZ[size]??SZ.md} ${V[variant]??V.default}`}>
            {key}
          </kbd>
        </>
      ))}
    </span>
  )
}

export default KBD
```

---

### List Group

```jsx
// src/components/ui/ListGroup.jsx
/**
 * ListGroup — Bordered list of items.
 *
 * Props:
 *   items   — [{ id, label, description?, icon?, badge?, meta?, onClick?, active?, disabled? }]
 *   variant — 'default'|'flush'|'separated'
 *   selectable — boolean
 *   numbered   — boolean
 */
export function ListGroup({ items = [], variant = 'default', selectable = false, numbered = false }) {
  const wrap = {
    default:   'border border-grey-200 dark:border-grey-700 rounded-xl overflow-hidden divide-y divide-grey-200 dark:divide-grey-700',
    flush:     'divide-y divide-grey-200 dark:divide-grey-700',
    separated: 'space-y-2',
  }

  return (
    <ul className={`font-aumovio ${wrap[variant]??wrap.default}`} role="list">
      {items.map((item, i) => (
        <li key={item.id ?? i}
          onClick={() => !item.disabled && item.onClick?.()}
          className={`flex items-center gap-3 px-4 py-3 bg-white dark:bg-[#1a1030] text-sm
            transition-all duration-150
            ${variant === 'separated' ? 'rounded-xl border border-grey-200 dark:border-grey-700' : ''}
            ${selectable && !item.disabled ? 'cursor-pointer hover:bg-orange-50 dark:hover:bg-orange-400/5 hover:text-orange-400' : ''}
            ${item.active ? 'bg-orange-50 dark:bg-orange-400/10 text-orange-400 border-l-4 border-orange-400' : ''}
            ${item.disabled ? 'opacity-40 cursor-not-allowed' : ''}`}>
          {numbered && (
            <span className="w-6 h-6 rounded-full bg-orange-400/10 text-orange-400 text-xs
              font-aumovio-bold flex items-center justify-center shrink-0">
              {i + 1}
            </span>
          )}
          {item.icon && <item.icon className="w-4 h-4 text-grey-400 shrink-0" />}
          <div className="flex-1 min-w-0">
            <p className={`font-aumovio-bold truncate ${item.active ? 'text-orange-400' : 'text-black/85 dark:text-white/85'}`}>
              {item.label}
            </p>
            {item.description && (
              <p className="text-xs text-grey-500 dark:text-grey-400 truncate mt-0.5">{item.description}</p>
            )}
          </div>
          {item.meta && <span className="text-xs text-grey-400 shrink-0">{item.meta}</span>}
          {item.badge && item.badge}
        </li>
      ))}
    </ul>
  )
}

export default ListGroup
```

---

### Modal

```jsx
// src/components/ui/Modal.jsx
/**
 * Modal — Accessible dialog overlay.
 *
 * Props:
 *   open     — boolean
 *   onClose  — () => void
 *   title    — string
 *   size     — 'sm'|'md'|'lg'|'xl'|'2xl'|'full'
 *   variant  — 'default'|'danger'|'success'
 *   footer   — ReactNode
 *   children
 */
import { useEffect } from 'react'
import { XMarkIcon } from '@heroicons/react/24/outline'

const SIZES = {
  sm:'max-w-sm', md:'max-w-md', lg:'max-w-lg',
  xl:'max-w-2xl', '2xl':'max-w-4xl', full:'max-w-full min-h-screen rounded-none'
}

const VARIANTS = {
  default: '',
  danger:  'border-t-4 border-danger-400',
  success: 'border-t-4 border-success-400',
}

export function Modal({ open, onClose, title, size = 'md', variant = 'default', footer, children }) {
  useEffect(() => {
    if (!open) return
    const h = (e) => { if (e.key === 'Escape') onClose?.() }
    document.addEventListener('keydown', h)
    document.body.style.overflow = 'hidden'
    return () => { document.removeEventListener('keydown', h); document.body.style.overflow = '' }
  }, [open, onClose])

  if (!open) return null

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center p-4" role="dialog" aria-modal="true">
      <div className="absolute inset-0 bg-black/50 backdrop-blur-sm animate-fade-in"
        onClick={onClose} />
      <div className={`relative w-full ${SIZES[size]??SIZES.md}
        bg-white dark:bg-[#1a1030] rounded-2xl shadow-2xl
        animate-scale-in overflow-hidden font-aumovio ${VARIANTS[variant]}`}>
        {/* Header */}
        {title && (
          <div className="flex items-center justify-between px-6 py-4 border-b border-grey-200 dark:border-grey-700">
            <h2 className="font-aumovio-bold text-base text-black/85 dark:text-white/90">{title}</h2>
            <button onClick={onClose} aria-label="Close"
              className="flex items-center justify-center w-7 h-7 rounded-lg text-grey-400
                hover:text-grey-600 hover:bg-grey-100 dark:hover:bg-grey-800 transition-colors">
              <XMarkIcon className="w-4 h-4" />
            </button>
          </div>
        )}
        {/* Body */}
        <div className="px-6 py-5 overflow-y-auto max-h-[70vh]">{children}</div>
        {/* Footer */}
        {footer && (
          <div className="px-6 py-4 border-t border-grey-200 dark:border-grey-700
            bg-grey-50 dark:bg-white/5 flex items-center justify-end gap-3">
            {footer}
          </div>
        )}
      </div>
    </div>
  )
}

export default Modal
```

---

### Pagination

```jsx
// src/components/ui/Pagination.jsx
/**
 * Pagination — Page navigation controls.
 *
 * Props:
 *   page       — current page (1-indexed)
 *   totalPages — total number of pages
 *   onChange   — (page: number) => void
 *   siblingCount — pages shown on each side of current (default 1)
 *   showEnds   — boolean (first/last page buttons)
 *   size       — 'sm'|'md'|'lg'
 *   variant    — 'default'|'minimal'|'rounded'
 */
import { ChevronLeftIcon, ChevronRightIcon } from '@heroicons/react/24/outline'

function buildRange(start, end) {
  return Array.from({ length: end - start + 1 }, (_, i) => start + i)
}

function getPages(current, total, sibling = 1) {
  const range = sibling + 5
  if (total <= range) return buildRange(1, total)
  const left  = Math.max(current - sibling, 1)
  const right = Math.min(current + sibling, total)
  const showLeft  = left  > 2
  const showRight = right < total - 1
  const pages = []
  pages.push(1)
  if (showLeft) pages.push('...')
  pages.push(...buildRange(left, right))
  if (showRight) pages.push('...')
  pages.push(total)
  return [...new Set(pages)]
}

const SZ = { sm:'w-7 h-7 text-xs', md:'w-8 h-8 text-sm', lg:'w-10 h-10 text-base' }

export function Pagination({ page, totalPages, onChange, siblingCount = 1, showEnds = true, size = 'md', variant = 'default' }) {
  if (totalPages <= 1) return null
  const pages = getPages(page, totalPages, siblingCount)
  const sz = SZ[size] ?? SZ.md
  const radius = variant === 'rounded' ? 'rounded-full' : 'rounded-lg'

  const btn = (label, targetPage, disabled, icon) => (
    <button
      key={typeof label === 'string' ? label : undefined}
      onClick={() => !disabled && onChange?.(targetPage)}
      disabled={disabled}
      aria-label={typeof label === 'string' ? label : undefined}
      aria-current={label === page ? 'page' : undefined}
      className={`flex items-center justify-center font-aumovio-bold shrink-0
        border transition-all duration-200 ${sz} ${radius}
        ${disabled ? 'opacity-40 cursor-not-allowed' : 'cursor-pointer'}
        ${label === page
          ? 'bg-orange-400 text-white border-orange-400 shadow-lg shadow-orange-400/30'
          : label === '...'
            ? 'bg-transparent border-transparent text-grey-400 cursor-default hover:bg-transparent'
            : 'bg-white dark:bg-[#1a1030] border-grey-200 dark:border-grey-700 text-black/70 dark:text-white/70 hover:border-orange-400 hover:text-orange-400 dark:hover:border-orange-400'
        }`}>
      {icon ?? label}
    </button>
  )

  return (
    <nav aria-label="Pagination" className="flex items-center gap-1 font-aumovio flex-wrap">
      {btn('Prev', page - 1, page <= 1, <ChevronLeftIcon className="w-4 h-4" />)}
      {!showEnds
        ? null
        : pages.map((p, i) => btn(p, typeof p === 'number' ? p : page, p === '...', null))}
      {btn('Next', page + 1, page >= totalPages, <ChevronRightIcon className="w-4 h-4" />)}
    </nav>
  )
}

export default Pagination
```

---

### Popover

```jsx
// src/components/ui/Popover.jsx
/**
 * Popover — Rich floating panel anchored to a trigger.
 *
 * Props:
 *   trigger   — ReactNode
 *   content   — ReactNode
 *   placement — 'top'|'bottom'|'left'|'right'
 *   title     — string (optional header)
 *   width     — 'sm'|'md'|'lg'
 *   trigger   — 'click'|'hover'
 */
import { useState, useRef, useEffect } from 'react'

const PLACEMENT = {
  top:    'bottom-full left-1/2 -translate-x-1/2 mb-2',
  bottom: 'top-full  left-1/2 -translate-x-1/2 mt-2',
  left:   'right-full top-1/2 -translate-y-1/2 mr-2',
  right:  'left-full  top-1/2 -translate-y-1/2 ml-2',
}

const WIDTHS = { sm:'w-48', md:'w-64', lg:'w-80' }

export function Popover({ trigger, content, title, placement = 'bottom', width = 'md', triggerOn = 'click' }) {
  const [open, setOpen] = useState(false)
  const ref = useRef(null)

  useEffect(() => {
    if (triggerOn !== 'click') return
    const h = (e) => { if (!ref.current?.contains(e.target)) setOpen(false) }
    document.addEventListener('mousedown', h)
    return () => document.removeEventListener('mousedown', h)
  }, [triggerOn])

  const handlers = triggerOn === 'hover'
    ? { onMouseEnter: () => setOpen(true), onMouseLeave: () => setOpen(false) }
    : { onClick: () => setOpen(o => !o) }

  return (
    <div ref={ref} className="relative inline-block font-aumovio" {...handlers}>
      {trigger}
      {open && (
        <div className={`absolute z-50 ${PLACEMENT[placement]} ${WIDTHS[width]}
          bg-white dark:bg-[#1a1030] border border-grey-200 dark:border-grey-700
          rounded-xl shadow-2xl animate-scale-in`}>
          {title && (
            <div className="px-4 py-3 border-b border-grey-200 dark:border-grey-700">
              <h3 className="font-aumovio-bold text-sm text-black/85 dark:text-white/90">{title}</h3>
            </div>
          )}
          <div className="p-4 text-sm text-black/70 dark:text-white/70">{content}</div>
        </div>
      )}
    </div>
  )
}

export default Popover
```

---

### Progress

```jsx
// src/components/ui/Progress.jsx
/**
 * Progress — Linear or circular progress indicator.
 *
 * Props:
 *   value     — 0-100
 *   max       — number (default 100)
 *   variant   — 'primary'|'success'|'danger'|'warning'|'gradient'
 *   size      — 'xs'|'sm'|'md'|'lg'
 *   label     — boolean (show percentage)
 *   animated  — boolean (striped animation)
 *   circular  — boolean (ring variant)
 *   radius    — number (for circular, default 36)
 */
const COLORS = {
  primary:  'bg-orange-400',
  success:  'bg-success-400',
  danger:   'bg-danger-400',
  warning:  'bg-warn-400',
  gradient: 'bg-gradient-to-r from-orange-400 to-purple-400',
  purple:   'bg-purple-400',
}

const HEIGHTS = { xs:'h-1', sm:'h-1.5', md:'h-2.5', lg:'h-4' }

export function Progress({ value = 0, max = 100, variant = 'primary', size = 'md',
  label = false, animated = false, circular = false, radius = 36 }) {
  const pct = Math.min(100, Math.max(0, (value / max) * 100))

  if (circular) {
    const circ = 2 * Math.PI * radius
    const dash = circ - (pct / 100) * circ
    const sz = radius * 2 + 20
    return (
      <div className="relative inline-flex items-center justify-center font-aumovio">
        <svg width={sz} height={sz} className="-rotate-90">
          <circle cx={sz/2} cy={sz/2} r={radius}
            fill="none" stroke="#F0F0F0" strokeWidth="8" />
          <circle cx={sz/2} cy={sz/2} r={radius}
            fill="none" stroke="#FF4208" strokeWidth="8"
            strokeDasharray={circ} strokeDashoffset={dash}
            strokeLinecap="round"
            style={{ transition: 'stroke-dashoffset 0.6s ease' }} />
        </svg>
        <span className="absolute text-sm font-aumovio-bold text-black/85 dark:text-white/90">
          {Math.round(pct)}%
        </span>
      </div>
    )
  }

  return (
    <div className="w-full font-aumovio">
      {label && (
        <div className="flex justify-between mb-1">
          <span className="text-xs text-grey-500 dark:text-grey-400">Progress</span>
          <span className="text-xs font-aumovio-bold text-black/70 dark:text-white/70">{Math.round(pct)}%</span>
        </div>
      )}
      <div className={`w-full bg-grey-200 dark:bg-grey-700 rounded-full overflow-hidden ${HEIGHTS[size]??HEIGHTS.md}`}>
        <div
          className={`h-full rounded-full ${COLORS[variant]??COLORS.primary}
            transition-all duration-700 ease-out
            ${animated ? 'bg-stripes animate-stripes' : ''}`}
          style={{ width: `${pct}%` }}
          role="progressbar" aria-valuenow={value} aria-valuemax={max}
        />
      </div>
    </div>
  )
}

export default Progress
```

---

### Rating

```jsx
// src/components/ui/Rating.jsx
/**
 * Rating — Star rating input or display.
 *
 * Props:
 *   value     — number (0-max)
 *   max       — number (default 5)
 *   onChange  — (val: number) => void (if omitted = read-only)
 *   size      — 'sm'|'md'|'lg'
 *   color     — 'orange'|'yellow'|'purple'
 *   half      — boolean (allow 0.5 steps)
 *   showValue — boolean
 */
import { useState } from 'react'

const COLORS = { orange:'text-orange-400', yellow:'text-yellow-400', purple:'text-purple-400' }
const SZ = { sm:'w-4 h-4',md:'w-5 h-5',lg:'w-7 h-7' }

export function Rating({ value = 0, max = 5, onChange, size = 'md', color = 'orange', showValue = false }) {
  const [hover, setHover] = useState(null)
  const readOnly = !onChange
  const display = hover ?? value
  const sz = SZ[size] ?? SZ.md
  const col = COLORS[color] ?? COLORS.orange

  return (
    <div className="inline-flex items-center gap-0.5 font-aumovio">
      {Array.from({ length: max }, (_, i) => i + 1).map((star) => (
        <button
          key={star}
          type="button"
          disabled={readOnly}
          onClick={() => onChange?.(star)}
          onMouseEnter={() => !readOnly && setHover(star)}
          onMouseLeave={() => !readOnly && setHover(null)}
          className={`transition-all duration-150 ${readOnly ? 'cursor-default' : 'cursor-pointer hover:scale-110'}`}
          aria-label={`${star} star${star !== 1 ? 's' : ''}`}
        >
          <svg className={`${sz} transition-colors ${display >= star ? col : 'text-grey-200 dark:text-grey-700'}`}
            viewBox="0 0 20 20" fill="currentColor">
            <path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"/>
          </svg>
        </button>
      ))}
      {showValue && (
        <span className="ml-1.5 text-sm font-aumovio-bold text-black/60 dark:text-white/60">
          {value.toFixed(1)}
        </span>
      )}
    </div>
  )
}

export default Rating
```

---

### Skeleton

```jsx
// src/components/ui/Skeleton.jsx
/**
 * Skeleton — Loading placeholder.
 *
 * Props:
 *   variant  — 'text'|'rect'|'circle'|'card'|'table'|'list'
 *   width    — string (CSS, default '100%')
 *   height   — string (CSS, default auto)
 *   lines    — number (for text variant)
 *   count    — number (repeat)
 *   animate  — boolean
 */
const base = 'bg-grey-200 dark:bg-grey-700 rounded animate-pulse'

function SkeletonLine({ w = '100%', h = '0.875rem' }) {
  return <div className={base} style={{ width: w, height: h }} />
}

export function Skeleton({ variant = 'text', width = '100%', height, lines = 3, count = 1, animate = true }) {
  const pulse = animate ? '' : '!animate-none'

  if (variant === 'circle') return (
    <div className={`${base} ${pulse} rounded-full`}
      style={{ width: width, height: width }} />
  )

  if (variant === 'text') return (
    <div className="space-y-2" style={{ width }}>
      {Array.from({ length: lines }, (_, i) => (
        <SkeletonLine key={i} w={i === lines - 1 && lines > 1 ? '70%' : '100%'} />
      ))}
    </div>
  )

  if (variant === 'card') return (
    <div className="bg-white dark:bg-[#1a1030] border border-grey-200 dark:border-grey-700 rounded-xl overflow-hidden p-5 space-y-4" style={{ width }}>
      <div className={`${base} ${pulse} h-36 rounded-lg`} />
      <SkeletonLine w="60%" h="1.125rem" />
      <SkeletonLine />
      <SkeletonLine w="80%" />
    </div>
  )

  if (variant === 'list') return (
    <div className="space-y-3" style={{ width }}>
      {Array.from({ length: lines }, (_, i) => (
        <div key={i} className="flex items-center gap-3">
          <div className={`${base} ${pulse} rounded-full shrink-0`} style={{ width: 36, height: 36 }} />
          <div className="flex-1 space-y-1.5">
            <SkeletonLine w="50%" />
            <SkeletonLine w="80%" h="0.75rem" />
          </div>
        </div>
      ))}
    </div>
  )

  if (variant === 'table') return (
    <div className="space-y-2" style={{ width }}>
      <div className="flex gap-4">
        {[3,5,5,3].map((w,i) => <SkeletonLine key={i} w={`${w}rem`} h="1rem" />)}
      </div>
      {Array.from({ length: lines }, (_, i) => (
        <div key={i} className="flex gap-4">
          {[3,5,5,3].map((w,j) => <SkeletonLine key={j} w={`${w}rem`} />)}
        </div>
      ))}
    </div>
  )

  return (
    <div className={`${base} ${pulse}`}
      style={{ width, height: height ?? '1rem' }} />
  )
}

export default Skeleton
```

---

### Speed Dial

```jsx
// src/components/ui/SpeedDial.jsx
/**
 * SpeedDial — Floating action button with expandable sub-actions.
 *
 * Props:
 *   icon       — main button icon component
 *   actions    — [{ id, label, icon, onClick, color? }]
 *   direction  — 'up'|'down'|'left'|'right'
 *   position   — 'bottom-right'|'bottom-left'|'top-right'|'top-left'
 *   tooltip    — boolean (show labels)
 */
import { useState } from 'react'
import { PlusIcon } from '@heroicons/react/24/outline'

const POSITIONS = {
  'bottom-right': 'fixed bottom-6 right-6 z-50',
  'bottom-left':  'fixed bottom-6 left-6  z-50',
  'top-right':    'fixed top-6 right-6 z-50',
  'top-left':     'fixed top-6 left-6  z-50',
}

export function SpeedDial({ icon: Icon = PlusIcon, actions = [], direction = 'up', position = 'bottom-right', tooltip = true }) {
  const [open, setOpen] = useState(false)
  const isV = direction === 'up' || direction === 'down'
  const isReverse = direction === 'up' || direction === 'left'

  const actionList = (
    <div className={`flex ${isV ? 'flex-col' : 'flex-row'} ${isReverse ? 'flex-col-reverse' : ''} gap-3 mb-3`}>
      {actions.map((a, i) => (
        <div key={a.id} className={`flex items-center gap-2 transition-all duration-200
          ${open ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-4 pointer-events-none'}`}
          style={{ transitionDelay: open ? `${i * 50}ms` : '0ms' }}>
          {tooltip && isReverse && (
            <span className="text-xs font-aumovio-bold bg-white dark:bg-grey-800
              text-black/80 dark:text-white/80 px-2 py-1 rounded-lg shadow border
              border-grey-200 dark:border-grey-700 whitespace-nowrap">
              {a.label}
            </span>
          )}
          <button
            onClick={() => { a.onClick?.(); setOpen(false) }}
            className={`w-10 h-10 rounded-full flex items-center justify-center shadow-lg
              transition-all duration-200 hover:scale-110 active:scale-95 text-white
              ${a.color ?? 'bg-purple-400 hover:bg-purple-500'}`}
            aria-label={a.label}>
            <a.icon className="w-5 h-5" />
          </button>
        </div>
      ))}
    </div>
  )

  return (
    <div className={`${POSITIONS[position]} flex flex-col items-center`}>
      {isReverse && actionList}
      <button
        onClick={() => setOpen(o => !o)}
        className={`w-14 h-14 rounded-full bg-orange-400 text-white shadow-2xl shadow-orange-400/40
          flex items-center justify-center transition-all duration-300
          hover:scale-110 active:scale-95 hover:bg-orange-500 focus-visible:outline-none
          focus-visible:ring-4 focus-visible:ring-orange-400/40
          ${open ? 'rotate-45' : ''}`}
        aria-label={open ? 'Close menu' : 'Open menu'}
        aria-expanded={open}>
        <Icon className="w-6 h-6" />
      </button>
      {!isReverse && actionList}
    </div>
  )
}

export default SpeedDial
```

---

### Spinner

```jsx
// src/components/ui/Spinner.jsx
/**
 * Spinner — Loading indicator variants.
 *
 * Props:
 *   size    — 'xs'|'sm'|'md'|'lg'|'xl'
 *   variant — 'ring'|'dots'|'bars'|'pulse'
 *   color   — 'primary'|'white'|'grey'
 *   label   — string
 *   fullPage — boolean
 */
const SZ = { xs:'w-4 h-4 border-2', sm:'w-5 h-5 border-2', md:'w-8 h-8 border-[3px]', lg:'w-12 h-12 border-4', xl:'w-16 h-16 border-[5px]' }
const COL = { primary:'border-orange-400 border-t-transparent', white:'border-white border-t-transparent', grey:'border-grey-400 border-t-transparent' }

export function Spinner({ size = 'md', variant = 'ring', color = 'primary', label, fullPage = false }) {
  let element

  if (variant === 'ring') {
    element = (
      <div className="flex flex-col items-center gap-3">
        <div className={`${SZ[size]??SZ.md} ${COL[color]??COL.primary} rounded-full animate-spin`} />
        {label && <p className="text-sm font-aumovio text-grey-400 animate-pulse">{label}</p>}
      </div>
    )
  } else if (variant === 'dots') {
    const dotCol = color === 'primary' ? 'bg-orange-400' : color === 'white' ? 'bg-white' : 'bg-grey-400'
    element = (
      <div className="flex gap-1.5 items-center">
        {[0,1,2].map(i => (
          <div key={i} className={`w-2 h-2 rounded-full ${dotCol} animate-bounce`}
            style={{ animationDelay: `${i * 0.15}s` }} />
        ))}
      </div>
    )
  } else if (variant === 'pulse') {
    const pulseCol = color === 'primary' ? 'bg-orange-400' : color === 'white' ? 'bg-white' : 'bg-grey-400'
    element = <div className={`${SZ[size]??SZ.md} ${pulseCol} rounded-full animate-ping opacity-75`} />
  }

  if (fullPage) return (
    <div className="flex items-center justify-center min-h-screen bg-white dark:bg-[#0D0D14]">
      {element}
    </div>
  )

  return (
    <div className="flex items-center justify-center py-8">{element}</div>
  )
}

export default Spinner
```

---

### Stepper

```jsx
// src/components/ui/Stepper.jsx
/**
 * Stepper — Multi-step wizard progress indicator.
 *
 * Props:
 *   steps   — [{ id, label, description?, icon? }]
 *   current — current step index (0-based)
 *   variant — 'default'|'numbered'|'icon'
 *   orientation — 'horizontal'|'vertical'
 */
import { CheckIcon } from '@heroicons/react/24/outline'

export function Stepper({ steps = [], current = 0, variant = 'numbered', orientation = 'horizontal' }) {
  const isV = orientation === 'vertical'

  return (
    <div className={`font-aumovio flex ${isV ? 'flex-col gap-0' : 'flex-row items-center gap-0'}`}>
      {steps.map((step, i) => {
        const done    = i < current
        const active  = i === current
        const pending = i > current

        return (
          <div key={step.id ?? i} className={`flex ${isV ? 'flex-row gap-4' : 'flex-col items-center'} flex-1 last:flex-none`}>
            <div className={`flex ${isV ? 'flex-col items-center' : 'flex-row items-center w-full'}`}>
              {/* Circle */}
              <div className={`flex items-center justify-center rounded-full font-aumovio-bold shrink-0
                transition-all duration-300 z-10
                ${done   ? 'w-8 h-8 bg-orange-400 text-white shadow-lg shadow-orange-400/30' : ''}
                ${active ? 'w-8 h-8 bg-orange-400 text-white ring-4 ring-orange-400/30 shadow-lg shadow-orange-400/30' : ''}
                ${pending ? 'w-8 h-8 bg-grey-100 dark:bg-grey-800 text-grey-400 border-2 border-grey-300 dark:border-grey-600' : ''}`}>
                {done
                  ? <CheckIcon className="w-4 h-4" />
                  : step.icon && variant === 'icon'
                    ? <step.icon className="w-4 h-4" />
                    : <span className="text-xs">{i + 1}</span>}
              </div>

              {/* Connector */}
              {i < steps.length - 1 && (
                <div className={`flex-1 transition-all duration-500
                  ${isV ? 'w-0.5 h-8 mx-auto my-1' : 'h-0.5 mx-2'}
                  ${done || active ? 'bg-orange-400' : 'bg-grey-200 dark:bg-grey-700'}`} />
              )}
            </div>

            {/* Label */}
            <div className={`${isV ? 'pb-6 flex-1' : 'mt-2 text-center'} min-w-0`}>
              <p className={`text-xs font-aumovio-bold truncate
                ${active ? 'text-orange-400' : done ? 'text-black/70 dark:text-white/70' : 'text-grey-400'}`}>
                {step.label}
              </p>
              {step.description && (
                <p className="text-xs text-grey-400 mt-0.5 line-clamp-2">{step.description}</p>
              )}
            </div>
          </div>
        )
      })}
    </div>
  )
}

export default Stepper
```

---

### Tables

```jsx
// src/components/ui/Table.jsx
/**
 * Table — Data table with sort, pagination, selection, and empty state.
 *
 * Props:
 *   columns — [{ key, label, sortable?, render?: (row) => ReactNode, width? }]
 *   data    — array of row objects (each needs a unique `id`)
 *   loading — boolean
 *   selectable — boolean
 *   selectedIds — Set<id>
 *   onSelect — (id, checked) => void
 *   onSelectAll — (checked) => void
 *   sortKey    — string
 *   sortDir    — 'asc'|'desc'
 *   onSort     — (key) => void
 *   emptyText  — string
 *   stickyHeader — boolean
 *   striped    — boolean
 *   compact    — boolean
 */
import { ChevronUpIcon, ChevronDownIcon } from '@heroicons/react/24/outline'
import Skeleton from './Skeleton'

export function Table({
  columns = [], data = [], loading = false,
  selectable = false, selectedIds = new Set(),
  onSelect, onSelectAll,
  sortKey, sortDir = 'asc', onSort,
  emptyText = 'No records found.',
  stickyHeader = false, striped = false, compact = false,
}) {
  const allSelected = data.length > 0 && data.every(r => selectedIds.has(r.id))
  const someSelected = data.some(r => selectedIds.has(r.id))
  const cellPad = compact ? 'px-4 py-2' : 'px-5 py-3.5'

  return (
    <div className="w-full overflow-x-auto rounded-xl border border-grey-200 dark:border-grey-700 font-aumovio">
      <table className="w-full text-sm">
        <thead className={`${stickyHeader ? 'sticky top-0 z-10' : ''}
          bg-grey-50 dark:bg-grey-800 border-b border-grey-200 dark:border-grey-700`}>
          <tr>
            {selectable && (
              <th className={`${cellPad} w-10`}>
                <input type="checkbox"
                  checked={allSelected}
                  ref={el => { if (el) el.indeterminate = someSelected && !allSelected }}
                  onChange={e => onSelectAll?.(e.target.checked)}
                  className="accent-orange-400 w-4 h-4 rounded cursor-pointer" />
              </th>
            )}
            {columns.map(col => (
              <th key={col.key}
                className={`${cellPad} text-left font-aumovio-bold text-grey-500 dark:text-grey-400
                  uppercase tracking-wider text-xs whitespace-nowrap
                  ${col.sortable ? 'cursor-pointer hover:text-orange-400 select-none' : ''}
                  ${col.width ?? ''}`}
                onClick={() => col.sortable && onSort?.(col.key)}>
                <span className="flex items-center gap-1">
                  {col.label}
                  {col.sortable && (
                    <span className="flex flex-col -space-y-0.5">
                      <ChevronUpIcon className={`w-2.5 h-2.5 ${sortKey === col.key && sortDir === 'asc' ? 'text-orange-400' : 'text-grey-300'}`} />
                      <ChevronDownIcon className={`w-2.5 h-2.5 ${sortKey === col.key && sortDir === 'desc' ? 'text-orange-400' : 'text-grey-300'}`} />
                    </span>
                  )}
                </span>
              </th>
            ))}
          </tr>
        </thead>
        <tbody className="divide-y divide-grey-100 dark:divide-grey-800">
          {loading
            ? Array.from({ length: 5 }, (_, i) => (
                <tr key={i} className="bg-white dark:bg-[#1a1030]">
                  {selectable && <td className={cellPad}><Skeleton variant="rect" width="1rem" height="1rem" /></td>}
                  {columns.map(col => (
                    <td key={col.key} className={cellPad}>
                      <Skeleton variant="rect" height="0.875rem" />
                    </td>
                  ))}
                </tr>
              ))
            : data.length === 0
              ? (
                  <tr>
                    <td colSpan={columns.length + (selectable ? 1 : 0)}
                      className="text-center py-16 text-grey-400 dark:text-grey-500">
                      {emptyText}
                    </td>
                  </tr>
                )
              : data.map((row, ri) => {
                  const isSelected = selectedIds.has(row.id)
                  return (
                    <tr key={row.id ?? ri}
                      className={`transition-colors duration-150
                        ${striped && ri % 2 === 1 ? 'bg-grey-50/50 dark:bg-grey-800/30' : 'bg-white dark:bg-[#1a1030]'}
                        ${isSelected ? 'bg-orange-50 dark:bg-orange-400/5' : ''}
                        hover:bg-orange-50/60 dark:hover:bg-orange-400/5`}>
                      {selectable && (
                        <td className={cellPad}>
                          <input type="checkbox"
                            checked={isSelected}
                            onChange={e => onSelect?.(row.id, e.target.checked)}
                            className="accent-orange-400 w-4 h-4 cursor-pointer" />
                        </td>
                      )}
                      {columns.map(col => (
                        <td key={col.key} className={`${cellPad} text-black/75 dark:text-white/75`}>
                          {col.render ? col.render(row) : row[col.key] ?? '—'}
                        </td>
                      ))}
                    </tr>
                  )
                })
          }
        </tbody>
      </table>
    </div>
  )
}

export default Table
```

---

### Tabs

```jsx
// src/components/ui/Tabs.jsx
/**
 * Tabs — Tabbed content navigation.
 *
 * Props:
 *   tabs     — [{ id, label, icon?, badge?, disabled?, content: ReactNode }]
 *   defaultTab — id
 *   variant  — 'underline'|'pill'|'boxed'|'vertical'
 *   size     — 'sm'|'md'|'lg'
 *   fullWidth — boolean
 */
import { useState } from 'react'

const SZ = { sm:'px-3 py-1.5 text-xs', md:'px-4 py-2 text-sm', lg:'px-5 py-3 text-base' }

export function Tabs({ tabs = [], defaultTab, variant = 'underline', size = 'md', fullWidth = false }) {
  const [active, setActive] = useState(defaultTab ?? tabs[0]?.id)
  const activeTab = tabs.find(t => t.id === active)
  const sz = SZ[size] ?? SZ.md

  const NAV_STYLES = {
    underline: 'border-b border-grey-200 dark:border-grey-700 flex gap-1',
    pill:      'flex gap-1 p-1 bg-grey-100 dark:bg-grey-800 rounded-xl w-fit',
    boxed:     'flex gap-0 border border-grey-200 dark:border-grey-700 rounded-xl overflow-hidden w-fit',
    vertical:  'flex flex-col gap-1 border-r border-grey-200 dark:border-grey-700 pr-2',
  }

  const TAB_STYLES = {
    underline: (a) => `border-b-2 transition-all duration-200 font-aumovio-bold
      ${a ? 'border-orange-400 text-orange-400' : 'border-transparent text-grey-500 hover:text-orange-400 hover:border-orange-200'}`,
    pill: (a) => `rounded-lg transition-all duration-200 font-aumovio-bold
      ${a ? 'bg-white dark:bg-[#1a1030] text-orange-400 shadow-sm' : 'text-grey-500 hover:text-orange-400'}`,
    boxed: (a) => `border-r last:border-0 border-grey-200 dark:border-grey-700 font-aumovio-bold
      transition-all duration-200
      ${a ? 'bg-orange-400 text-white' : 'bg-white dark:bg-[#1a1030] text-grey-500 hover:bg-orange-50 hover:text-orange-400'}`,
    vertical: (a) => `rounded-lg text-left transition-all duration-200 font-aumovio-bold
      ${a ? 'bg-orange-50 dark:bg-orange-400/10 text-orange-400' : 'text-grey-500 hover:bg-grey-100 dark:hover:bg-grey-800 hover:text-orange-400'}`,
  }

  return (
    <div className={`font-aumovio ${variant === 'vertical' ? 'flex gap-6' : ''}`}>
      <div className={NAV_STYLES[variant]}>
        {tabs.map(tab => (
          <button
            key={tab.id}
            onClick={() => !tab.disabled && setActive(tab.id)}
            disabled={tab.disabled}
            className={`flex items-center gap-1.5 whitespace-nowrap
              disabled:opacity-40 disabled:cursor-not-allowed
              focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-orange-400/40 rounded
              ${sz} ${fullWidth ? 'flex-1 justify-center' : ''}
              ${(TAB_STYLES[variant] ?? TAB_STYLES.underline)(tab.id === active)}`}
          >
            {tab.icon && <tab.icon className="w-4 h-4 shrink-0" />}
            {tab.label}
            {tab.badge !== undefined && (
              <span className={`text-[10px] px-1.5 py-0.5 rounded-full font-aumovio-bold
                ${tab.id === active ? 'bg-white/20 text-white' : 'bg-orange-400/10 text-orange-400'}`}>
                {tab.badge}
              </span>
            )}
          </button>
        ))}
      </div>
      <div className={`${variant !== 'vertical' ? 'mt-4' : 'flex-1'}`}>
        {activeTab?.content}
      </div>
    </div>
  )
}

export default Tabs
```

---

### Timeline

```jsx
// src/components/ui/Timeline.jsx
/**
 * Timeline — Vertical list of events in time order.
 *
 * Props:
 *   items  — [{ id, title, description, date, icon?, color?, badge? }]
 *   variant — 'left'|'alternating'
 *   connect — boolean (connecting line)
 */
const COLORS = {
  orange:  'bg-orange-400  ring-orange-400/30',
  purple:  'bg-purple-400  ring-purple-400/30',
  success: 'bg-success-400 ring-success-400/30',
  danger:  'bg-danger-400  ring-danger-400/30',
  warn:    'bg-warn-400    ring-warn-400/30',
  blue:    'bg-blue-400    ring-blue-400/30',
  grey:    'bg-grey-400    ring-grey-400/30',
}

export function Timeline({ items = [], variant = 'left', connect = true }) {
  return (
    <ol className="relative font-aumovio">
      {connect && (
        <div className="absolute left-4 top-4 bottom-4 w-0.5 bg-grey-200 dark:bg-grey-700" />
      )}
      <div className="space-y-8">
        {items.map((item, i) => {
          const colCls = COLORS[item.color ?? 'orange'] ?? COLORS.orange
          return (
            <li key={item.id ?? i} className="relative flex gap-5 pl-2">
              {/* Dot */}
              <div className={`relative z-10 w-8 h-8 rounded-full shrink-0
                flex items-center justify-center ring-4 text-white shadow
                ${colCls}`}>
                {item.icon
                  ? <item.icon className="w-4 h-4" />
                  : <span className="w-2 h-2 rounded-full bg-white" />}
              </div>
              {/* Content */}
              <div className="flex-1 pb-2">
                <div className="flex items-center justify-between gap-3 mb-1 flex-wrap">
                  <h3 className="font-aumovio-bold text-sm text-black/85 dark:text-white/90">
                    {item.title}
                  </h3>
                  <div className="flex items-center gap-2">
                    {item.badge && item.badge}
                    <time className="text-xs text-grey-400">{item.date}</time>
                  </div>
                </div>
                {item.description && (
                  <p className="text-sm text-grey-500 dark:text-grey-400 leading-relaxed">
                    {item.description}
                  </p>
                )}
              </div>
            </li>
          )
        })}
      </div>
    </ol>
  )
}

export default Timeline
```

---

### Toast

> Use `react-toastify` directly as configured in `main.jsx`. Below is an optional custom implementation.

```jsx
// src/components/ui/toast.utils.js — thin wrappers
import { toast as t } from 'react-toastify'

export const toast = {
  success: (msg, opts) => t.success(msg, opts),
  error:   (msg, opts) => t.error(msg, opts),
  warning: (msg, opts) => t.warning(msg, opts),
  info:    (msg, opts) => t.info(msg, opts),
  loading: (msg, opts) => t.loading(msg, opts),
  dismiss: (id)        => t.dismiss(id),
  promise: (promise, { loading, success, error }, opts) =>
    t.promise(promise, { pending: loading, success, error }, opts),
}
```

**Toast config (main.jsx):**
```jsx
<ToastContainer
  position="bottom-right"
  autoClose={5000}
  hideProgressBar={false}
  closeOnClick
  pauseOnHover
  draggable
  theme="colored"
  className="z-[9999]"
/>
```

---

### Tooltips

```jsx
// src/components/ui/Tooltip.jsx
/**
 * Tooltip — Hover label attached to any element.
 *
 * Props:
 *   children  — trigger element
 *   content   — string | ReactNode
 *   placement — 'top'|'bottom'|'left'|'right'
 *   delay     — ms (default 300)
 *   size      — 'sm'|'md'
 *   disabled  — boolean
 */
import { useState, useRef } from 'react'

const PL = {
  top:    'bottom-full left-1/2 -translate-x-1/2 mb-2',
  bottom: 'top-full  left-1/2 -translate-x-1/2 mt-2',
  left:   'right-full top-1/2 -translate-y-1/2 mr-2',
  right:  'left-full  top-1/2 -translate-y-1/2 ml-2',
}

export function Tooltip({ children, content, placement = 'top', delay = 300, size = 'sm', disabled = false }) {
  const [visible, setVisible] = useState(false)
  const timer = useRef(null)

  const show = () => { if (disabled) return; timer.current = setTimeout(() => setVisible(true), delay) }
  const hide = () => { clearTimeout(timer.current); setVisible(false) }

  return (
    <div className="relative inline-flex" onMouseEnter={show} onMouseLeave={hide} onFocus={show} onBlur={hide}>
      {children}
      {visible && content && (
        <div className={`absolute z-50 pointer-events-none animate-fade-in whitespace-nowrap
          ${PL[placement] ?? PL.top}
          ${size === 'sm'
            ? 'px-2.5 py-1 text-xs rounded-lg'
            : 'px-3 py-1.5 text-sm rounded-xl'}
          bg-grey-900 dark:bg-grey-700 text-white font-aumovio shadow-xl`}>
          {content}
        </div>
      )}
    </div>
  )
}

export default Tooltip
```

---

### QR Code

```jsx
// src/components/ui/QRCode.jsx
/**
 * QRCode — Generates and renders a QR code.
 *
 * Props:
 *   value     — string (URL or text to encode)
 *   size      — number (px, default 160)
 *   level     — 'L'|'M'|'Q'|'H'
 *   bgColor   — string (default '#FFFFFF')
 *   fgColor   — string (default '#000000')
 *   logo      — string (URL for center logo)
 *   logoSize  — number (default 32)
 *   downloadable — boolean
 *   title     — string
 */
import { useRef } from 'react'
import { QRCodeSVG } from 'qrcode.react'
import Button from './Button'
import { ArrowDownTrayIcon } from '@heroicons/react/24/outline'

export function QRCode({
  value = '', size = 160, level = 'M',
  bgColor = '#FFFFFF', fgColor = '#000000',
  logo, logoSize = 32,
  downloadable = false, title,
}) {
  const ref = useRef(null)

  const download = () => {
    const svg = ref.current?.querySelector('svg')
    if (!svg) return
    const blob = new Blob([svg.outerHTML], { type: 'image/svg+xml' })
    const url = URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = url; a.download = 'qrcode.svg'; a.click()
    URL.revokeObjectURL(url)
  }

  return (
    <div className="inline-flex flex-col items-center gap-3 font-aumovio">
      {title && <p className="text-sm font-aumovio-bold text-black/70 dark:text-white/70">{title}</p>}
      <div ref={ref}
        className="p-3 bg-white rounded-xl shadow-lg border border-grey-200 dark:border-grey-300">
        <QRCodeSVG
          value={value || 'https://aumovio.com'}
          size={size}
          level={level}
          bgColor={bgColor}
          fgColor={fgColor}
          imageSettings={logo ? {
            src: logo, x: undefined, y: undefined,
            height: logoSize, width: logoSize, excavate: true,
          } : undefined}
        />
      </div>
      {downloadable && (
        <Button variant="ghost" size="sm" leftIcon={ArrowDownTrayIcon} onClick={download}>
          Download SVG
        </Button>
      )}
    </div>
  )
}

export default QRCode
```

---

## 7. Forms

### Input Field

```jsx
// src/components/forms/Input.jsx
/**
 * Input — Text input with label, helper, error, icons.
 *
 * Props:
 *   label, name, type, value, onChange, placeholder
 *   error       — string
 *   helper      — string
 *   leftIcon    — icon component
 *   rightElement — ReactNode (custom right slot)
 *   disabled, required, autoComplete
 *   size        — 'sm'|'md'|'lg'
 *   variant     — 'default'|'filled'|'underline'
 */
const SZ = {
  sm: 'px-3 py-1.5 text-xs rounded-lg',
  md: 'px-3.5 py-2 text-sm rounded-lg',
  lg: 'px-4 py-3 text-base rounded-xl',
}

export function Input({
  label, name, type = 'text', value, onChange, placeholder,
  error, helper, leftIcon: LeftIcon, rightElement,
  disabled = false, required = false, autoComplete,
  size = 'md', id,
}) {
  const inputId = id ?? name

  return (
    <div className="w-full font-aumovio">
      {label && (
        <label htmlFor={inputId}
          className="block text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-1.5">
          {label}{required && <span className="text-danger-400 ml-0.5">*</span>}
        </label>
      )}
      <div className="relative">
        {LeftIcon && (
          <span className="absolute left-3 top-1/2 -translate-y-1/2 text-grey-400 pointer-events-none">
            <LeftIcon className="w-4 h-4" />
          </span>
        )}
        <input
          id={inputId} name={name} type={type} value={value}
          onChange={onChange} placeholder={placeholder}
          disabled={disabled} required={required}
          autoComplete={autoComplete}
          className={`w-full bg-white dark:bg-[#1a1030] font-aumovio
            text-black/85 dark:text-white/85
            border transition-all duration-200
            placeholder-grey-400 dark:placeholder-grey-600
            focus:outline-none focus:ring-2 focus:shadow-md
            disabled:opacity-50 disabled:cursor-not-allowed
            ${LeftIcon ? 'pl-9' : ''} ${rightElement ? 'pr-10' : ''}
            ${SZ[size] ?? SZ.md}
            ${error
              ? 'border-danger-400 focus:ring-danger-400/30 focus:border-danger-400 bg-danger-100/10'
              : 'border-grey-300 dark:border-grey-700 focus:ring-orange-400/30 focus:border-orange-400'
            }`}
        />
        {rightElement && (
          <span className="absolute right-3 top-1/2 -translate-y-1/2">{rightElement}</span>
        )}
      </div>
      {error && <p className="mt-1.5 text-xs text-danger-400 font-aumovio-bold">{error}</p>}
      {!error && helper && <p className="mt-1.5 text-xs text-grey-400">{helper}</p>}
    </div>
  )
}

export default Input
```

---

### File Input

```jsx
// src/components/forms/FileInput.jsx
/**
 * FileInput — Styled file picker with drag-and-drop.
 *
 * Props:
 *   label, name, accept, multiple, disabled
 *   onChange    — (files: FileList) => void
 *   maxSize     — bytes (for validation hint)
 *   preview     — boolean (image thumbnails)
 *   dropzone    — boolean (full drop zone variant)
 *   error
 */
import { useRef, useState } from 'react'
import { CloudArrowUpIcon, XMarkIcon } from '@heroicons/react/24/outline'

export function FileInput({
  label, name, accept, multiple = false, disabled = false,
  onChange, maxSize, error, preview = false, dropzone = true,
}) {
  const ref = useRef(null)
  const [files, setFiles] = useState([])
  const [dragging, setDragging] = useState(false)

  const handle = (incoming) => {
    const arr = Array.from(incoming)
    setFiles(multiple ? arr : arr.slice(0, 1))
    onChange?.(incoming)
  }

  const removeFile = (i) => {
    const next = files.filter((_, fi) => fi !== i)
    setFiles(next)
  }

  if (!dropzone) return (
    <div className="font-aumovio">
      {label && <label className="block text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-1.5">{label}</label>}
      <input type="file" name={name} accept={accept} multiple={multiple}
        disabled={disabled} onChange={e => handle(e.target.files)}
        className="block w-full text-sm text-grey-500 file:mr-4 file:py-2 file:px-4
          file:rounded-lg file:border-0 file:font-aumovio-bold file:text-xs
          file:bg-orange-400/10 file:text-orange-400 hover:file:bg-orange-400 hover:file:text-white
          file:cursor-pointer file:transition-all file:duration-200" />
    </div>
  )

  return (
    <div className="font-aumovio">
      {label && <label className="block text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-1.5">{label}</label>}
      <div
        onClick={() => !disabled && ref.current?.click()}
        onDragOver={e => { e.preventDefault(); !disabled && setDragging(true) }}
        onDragLeave={() => setDragging(false)}
        onDrop={e => { e.preventDefault(); setDragging(false); handle(e.dataTransfer.files) }}
        className={`relative border-2 border-dashed rounded-xl p-8 text-center cursor-pointer
          transition-all duration-200
          ${disabled ? 'opacity-50 cursor-not-allowed' : ''}
          ${error ? 'border-danger-400 bg-danger-100/10' : ''}
          ${dragging
            ? 'border-orange-400 bg-orange-50 dark:bg-orange-400/5 scale-[1.01]'
            : error ? '' : 'border-grey-300 dark:border-grey-700 hover:border-orange-400 bg-white dark:bg-[#1a1030]'
          }`}>
        <input ref={ref} type="file" name={name} accept={accept} multiple={multiple}
          className="sr-only" onChange={e => handle(e.target.files)} />
        <CloudArrowUpIcon className="w-10 h-10 mx-auto text-grey-300 dark:text-grey-600 mb-3" />
        <p className="text-sm font-aumovio-bold text-black/60 dark:text-white/60">
          Drop files here or <span className="text-orange-400">browse</span>
        </p>
        {accept && <p className="text-xs text-grey-400 mt-1">Accepted: {accept}</p>}
        {maxSize && <p className="text-xs text-grey-400">Max size: {(maxSize / 1024 / 1024).toFixed(1)} MB</p>}
      </div>
      {files.length > 0 && (
        <ul className="mt-3 space-y-2">
          {files.map((f, i) => (
            <li key={i} className="flex items-center gap-2 bg-grey-50 dark:bg-grey-800
              border border-grey-200 dark:border-grey-700 rounded-lg px-3 py-2 text-sm">
              {preview && f.type.startsWith('image/') && (
                <img src={URL.createObjectURL(f)} className="w-8 h-8 object-cover rounded" alt="" />
              )}
              <span className="flex-1 truncate text-black/70 dark:text-white/70">{f.name}</span>
              <span className="text-xs text-grey-400 shrink-0">{(f.size / 1024).toFixed(0)} KB</span>
              <button onClick={(e) => { e.stopPropagation(); removeFile(i) }}
                className="text-grey-400 hover:text-danger-400 transition-colors">
                <XMarkIcon className="w-4 h-4" />
              </button>
            </li>
          ))}
        </ul>
      )}
      {error && <p className="mt-1.5 text-xs text-danger-400 font-aumovio-bold">{error}</p>}
    </div>
  )
}

export default FileInput
```

---

### Search Input

```jsx
// src/components/forms/SearchInput.jsx
// (See full SearchBar in SearchBar.jsx — this is the standalone form version)
import { useCallback, useEffect, useRef, useState } from 'react'
import { MagnifyingGlassIcon, XMarkIcon } from '@heroicons/react/24/outline'

export function SearchInput({
  value = '', onChange, onSubmit, placeholder = 'Search…',
  disabled = false, loading = false,
  suggestions = [], onSuggestionSelect,
  debounce = 0, size = 'md',
}) {
  const [local, setLocal] = useState(value)
  const [showSug, setShowSug] = useState(false)
  const timer = useRef(null)
  const SZ = { sm:'py-1.5 text-xs pl-8 pr-8', md:'py-2 text-sm pl-9 pr-9', lg:'py-3 text-base pl-10 pr-10' }
  const ICON = { sm:'w-4 h-4 left-2', md:'w-4 h-4 left-3', lg:'w-5 h-5 left-3' }

  const emit = useCallback((v) => {
    clearTimeout(timer.current)
    if (debounce > 0) timer.current = setTimeout(() => onChange?.(v), debounce)
    else onChange?.(v)
  }, [onChange, debounce])

  useEffect(() => setLocal(value), [value])

  return (
    <div className="relative w-full font-aumovio">
      <MagnifyingGlassIcon className={`absolute top-1/2 -translate-y-1/2 ${ICON[size]} text-grey-400 pointer-events-none`} />
      <input
        type="search"
        value={local}
        onChange={e => { setLocal(e.target.value); emit(e.target.value); setShowSug(true) }}
        onKeyDown={e => { if (e.key === 'Enter') { onSubmit?.(local); setShowSug(false) } }}
        onBlur={() => setTimeout(() => setShowSug(false), 150)}
        placeholder={placeholder}
        disabled={disabled}
        className={`w-full rounded-xl border font-aumovio
          bg-white dark:bg-[#1a1030] text-black/85 dark:text-white/85 placeholder-grey-400
          border-grey-300 dark:border-grey-700
          focus:outline-none focus:ring-2 focus:ring-orange-400/30 focus:border-orange-400
          transition-all duration-200 disabled:opacity-50
          ${SZ[size] ?? SZ.md}`}
      />
      <span className="absolute right-3 top-1/2 -translate-y-1/2">
        {loading
          ? <span className="w-3.5 h-3.5 border-2 border-orange-400 border-t-transparent rounded-full animate-spin block" />
          : local && <button onClick={() => { setLocal(''); onChange?.('') }}>
              <XMarkIcon className="w-4 h-4 text-grey-400 hover:text-orange-400 transition-colors" />
            </button>}
      </span>
      {showSug && suggestions.length > 0 && (
        <ul className="absolute top-full left-0 right-0 mt-1 z-50 bg-white dark:bg-[#1a1030]
          border border-grey-200 dark:border-grey-700 rounded-xl shadow-2xl overflow-hidden">
          {suggestions.map((s, i) => (
            <li key={i} onMouseDown={() => { onSuggestionSelect?.(s); setLocal(s); setShowSug(false) }}
              className="px-4 py-2.5 text-sm cursor-pointer hover:bg-orange-50 dark:hover:bg-orange-400/5
                hover:text-orange-400 flex items-center gap-2">
              <MagnifyingGlassIcon className="w-3.5 h-3.5 text-grey-400 shrink-0" />
              {s}
            </li>
          ))}
        </ul>
      )}
    </div>
  )
}

export default SearchInput
```

---

### Number Input

```jsx
// src/components/forms/NumberInput.jsx
/**
 * NumberInput — Incrementable/decrementable numeric field.
 *
 * Props:
 *   value, onChange, min, max, step, label, error, disabled, size
 */
import { MinusIcon, PlusIcon } from '@heroicons/react/24/outline'

export function NumberInput({
  value = 0, onChange, min = -Infinity, max = Infinity, step = 1,
  label, error, disabled = false, size = 'md',
}) {
  const dec = () => { const n = Number(value) - step; if (n >= min) onChange?.(n) }
  const inc = () => { const n = Number(value) + step; if (n <= max) onChange?.(n) }
  const handle = (e) => {
    const n = parseFloat(e.target.value)
    if (!isNaN(n)) onChange?.(Math.min(max, Math.max(min, n)))
  }

  const SZ = { sm:'h-7 text-xs', md:'h-9 text-sm', lg:'h-11 text-base' }
  const BTN = { sm:'w-7', md:'w-9', lg:'w-11' }

  return (
    <div className="font-aumovio">
      {label && <label className="block text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-1.5">{label}</label>}
      <div className={`inline-flex items-center border rounded-xl overflow-hidden
        ${error ? 'border-danger-400' : 'border-grey-300 dark:border-grey-700'}
        bg-white dark:bg-[#1a1030]`}>
        <button onClick={dec} disabled={disabled || value <= min}
          className={`${SZ[size]??SZ.md} ${BTN[size]??BTN.md} flex items-center justify-center
            text-grey-500 hover:text-orange-400 hover:bg-orange-50 dark:hover:bg-orange-400/10
            disabled:opacity-40 disabled:cursor-not-allowed transition-colors border-r
            border-grey-200 dark:border-grey-700`}>
          <MinusIcon className="w-3.5 h-3.5" />
        </button>
        <input type="number" value={value} onChange={handle} disabled={disabled}
          min={min} max={max} step={step}
          className={`${SZ[size]??SZ.md} w-16 text-center font-aumovio-bold bg-transparent
            text-black/85 dark:text-white/85 border-0 focus:outline-none focus:ring-0
            [appearance:textfield] [&::-webkit-outer-spin-button]:appearance-none
            [&::-webkit-inner-spin-button]:appearance-none`} />
        <button onClick={inc} disabled={disabled || value >= max}
          className={`${SZ[size]??SZ.md} ${BTN[size]??BTN.md} flex items-center justify-center
            text-grey-500 hover:text-orange-400 hover:bg-orange-50 dark:hover:bg-orange-400/10
            disabled:opacity-40 disabled:cursor-not-allowed transition-colors border-l
            border-grey-200 dark:border-grey-700`}>
          <PlusIcon className="w-3.5 h-3.5" />
        </button>
      </div>
      {error && <p className="mt-1.5 text-xs text-danger-400 font-aumovio-bold">{error}</p>}
    </div>
  )
}

export default NumberInput
```

---

### Phone Input

```jsx
// src/components/forms/PhoneInput.jsx
/**
 * PhoneInput — International phone number input with country code.
 *
 * Props:
 *   value, onChange — combined "+63 912 345 6789" string
 *   label, error, disabled, size, placeholder
 *   countryCode — default '+63'
 */
import { useState } from 'react'

const CODES = [
  { flag:'🇵🇭', code:'+63', country:'PH' },
  { flag:'🇺🇸', code:'+1',  country:'US' },
  { flag:'🇬🇧', code:'+44', country:'GB' },
  { flag:'🇦🇺', code:'+61', country:'AU' },
  { flag:'🇨🇦', code:'+1',  country:'CA' },
  { flag:'🇯🇵', code:'+81', country:'JP' },
  { flag:'🇸🇬', code:'+65', country:'SG' },
]

export function PhoneInput({
  value = '', onChange, label, error, disabled = false, size = 'md',
  placeholder = '912 345 6789', countryCode = '+63',
}) {
  const [cc, setCc] = useState(countryCode)
  const num = value.startsWith(cc) ? value.slice(cc.length).trim() : value

  const emit = (country, number) => onChange?.(`${country} ${number}`.trim())

  const SZ = { sm:'py-1.5 text-xs', md:'py-2 text-sm', lg:'py-3 text-base' }

  return (
    <div className="font-aumovio">
      {label && <label className="block text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-1.5">{label}</label>}
      <div className={`flex rounded-xl border overflow-hidden bg-white dark:bg-[#1a1030]
        ${error ? 'border-danger-400' : 'border-grey-300 dark:border-grey-700 focus-within:border-orange-400 focus-within:ring-2 focus-within:ring-orange-400/30'}`}>
        <select
          value={cc}
          onChange={e => { setCc(e.target.value); emit(e.target.value, num) }}
          disabled={disabled}
          className={`bg-grey-50 dark:bg-grey-800 text-sm font-aumovio-bold
            border-r border-grey-200 dark:border-grey-700 px-2 focus:outline-none
            text-black/80 dark:text-white/80 cursor-pointer ${SZ[size]??SZ.md}`}>
          {CODES.map(c => (
            <option key={c.country} value={c.code}>{c.flag} {c.code}</option>
          ))}
        </select>
        <input
          type="tel" value={num} placeholder={placeholder} disabled={disabled}
          onChange={e => emit(cc, e.target.value)}
          className={`flex-1 px-3 font-aumovio bg-transparent text-black/85 dark:text-white/85
            placeholder-grey-400 focus:outline-none ${SZ[size]??SZ.md}`} />
      </div>
      {error && <p className="mt-1.5 text-xs text-danger-400 font-aumovio-bold">{error}</p>}
    </div>
  )
}

export default PhoneInput
```

---

### Select

```jsx
// src/components/forms/Select.jsx
/**
 * Select — Styled dropdown for option picking.
 *
 * Props:
 *   options  — [{ value, label, group? }]
 *   value, onChange, label, placeholder, error, disabled, multiple, size
 */
import { ChevronDownIcon } from '@heroicons/react/24/outline'

export function Select({
  options = [], value, onChange, label, placeholder = 'Select…',
  error, disabled = false, multiple = false, size = 'md', id, name,
}) {
  const inputId = id ?? name
  const SZ = { sm:'py-1.5 text-xs', md:'py-2 text-sm', lg:'py-3 text-base' }

  const groups = [...new Set(options.map(o => o.group).filter(Boolean))]
  const ungrouped = options.filter(o => !o.group)

  const renderOptions = (opts) => opts.map(o => (
    <option key={o.value} value={o.value}>{o.label}</option>
  ))

  return (
    <div className="font-aumovio">
      {label && (
        <label htmlFor={inputId}
          className="block text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-1.5">
          {label}
        </label>
      )}
      <div className="relative">
        <select
          id={inputId} name={name} value={value} multiple={multiple} disabled={disabled}
          onChange={e => onChange?.(multiple ? [...e.target.selectedOptions].map(o => o.value) : e.target.value)}
          className={`w-full rounded-xl border px-3 pr-8 font-aumovio appearance-none cursor-pointer
            bg-white dark:bg-[#1a1030] text-black/85 dark:text-white/85
            focus:outline-none focus:ring-2 focus:shadow-md transition-all duration-200
            disabled:opacity-50 disabled:cursor-not-allowed
            ${SZ[size]??SZ.md}
            ${error
              ? 'border-danger-400 focus:ring-danger-400/30'
              : 'border-grey-300 dark:border-grey-700 focus:ring-orange-400/30 focus:border-orange-400'
            }`}>
          {placeholder && !multiple && <option value="">{placeholder}</option>}
          {groups.length > 0
            ? <>
                {ungrouped.length > 0 && renderOptions(ungrouped)}
                {groups.map(g => (
                  <optgroup key={g} label={g}>
                    {renderOptions(options.filter(o => o.group === g))}
                  </optgroup>
                ))}
              </>
            : renderOptions(options)}
        </select>
        {!multiple && (
          <ChevronDownIcon className="absolute right-2.5 top-1/2 -translate-y-1/2 w-4 h-4 text-grey-400 pointer-events-none" />
        )}
      </div>
      {error && <p className="mt-1.5 text-xs text-danger-400 font-aumovio-bold">{error}</p>}
    </div>
  )
}

export default Select
```

---

### Textarea

```jsx
// src/components/forms/Textarea.jsx
/**
 * Textarea — Multi-line text input with auto-resize and char count.
 *
 * Props:
 *   label, name, value, onChange, placeholder, rows, maxLength
 *   error, helper, disabled, required, resize, showCount, size
 */
import { useRef, useEffect } from 'react'

export function Textarea({
  label, name, value = '', onChange, placeholder,
  rows = 4, maxLength, error, helper,
  disabled = false, required = false,
  resize = 'vertical', showCount = false, size = 'md', id,
}) {
  const inputId = id ?? name
  const ref = useRef(null)
  const SZ = { sm:'px-3 py-2 text-xs', md:'px-3.5 py-2.5 text-sm', lg:'px-4 py-3 text-base' }
  const RESIZE = { none:'resize-none', vertical:'resize-y', horizontal:'resize-x', both:'resize' }

  useEffect(() => {
    if (ref.current && resize === 'auto') {
      ref.current.style.height = 'auto'
      ref.current.style.height = ref.current.scrollHeight + 'px'
    }
  }, [value, resize])

  return (
    <div className="font-aumovio">
      {label && (
        <label htmlFor={inputId}
          className="block text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-1.5">
          {label}{required && <span className="text-danger-400 ml-0.5">*</span>}
        </label>
      )}
      <textarea
        ref={ref}
        id={inputId} name={name} value={value} rows={rows}
        onChange={onChange} placeholder={placeholder}
        disabled={disabled} required={required}
        maxLength={maxLength}
        className={`w-full rounded-xl border font-aumovio leading-relaxed
          bg-white dark:bg-[#1a1030] text-black/85 dark:text-white/85
          placeholder-grey-400 dark:placeholder-grey-600
          focus:outline-none focus:ring-2 focus:shadow-md transition-all duration-200
          disabled:opacity-50 disabled:cursor-not-allowed
          ${RESIZE[resize] ?? RESIZE.vertical}
          ${SZ[size]??SZ.md}
          ${error
            ? 'border-danger-400 focus:ring-danger-400/30'
            : 'border-grey-300 dark:border-grey-700 focus:ring-orange-400/30 focus:border-orange-400'}`}
      />
      <div className="flex justify-between mt-1.5">
        {error
          ? <p className="text-xs text-danger-400 font-aumovio-bold">{error}</p>
          : <p className="text-xs text-grey-400">{helper}</p>
        }
        {showCount && maxLength && (
          <span className={`text-xs font-aumovio-bold ${value.length >= maxLength ? 'text-danger-400' : 'text-grey-400'}`}>
            {value.length}/{maxLength}
          </span>
        )}
      </div>
    </div>
  )
}

export default Textarea
```

---

### Timepicker

```jsx
// src/components/forms/Timepicker.jsx
/**
 * Timepicker — Clock time selection.
 *
 * Props:
 *   value      — "HH:MM" string
 *   onChange   — (time: string) => void
 *   label, error, disabled, size
 *   use12Hour  — boolean
 *   minuteStep — 1|5|10|15|30
 */
import { useState, useRef, useEffect } from 'react'
import { ClockIcon } from '@heroicons/react/24/outline'

export function Timepicker({
  value = '', onChange, label, error, disabled = false, size = 'md',
  use12Hour = false, minuteStep = 5,
}) {
  const [open, setOpen] = useState(false)
  const [h, m, ampm] = (() => {
    const [hh = '12', mm = '00'] = (value || '').split(':')
    const hn = parseInt(hh)
    if (use12Hour) return [hn > 12 ? hn - 12 : hn || 12, mm, hn >= 12 ? 'PM' : 'AM']
    return [hn, mm, null]
  })()
  const ref = useRef(null)

  useEffect(() => {
    const handler = (e) => { if (!ref.current?.contains(e.target)) setOpen(false) }
    document.addEventListener('mousedown', handler)
    return () => document.removeEventListener('mousedown', handler)
  }, [])

  const emit = (newH, newM, newAmpm) => {
    let hour = newH
    if (use12Hour) {
      if (newAmpm === 'PM' && hour < 12) hour += 12
      if (newAmpm === 'AM' && hour === 12) hour = 0
    }
    onChange?.(`${String(hour).padStart(2,'0')}:${newM}`)
  }

  const hours = Array.from({ length: use12Hour ? 12 : 24 }, (_, i) => use12Hour ? i + 1 : i)
  const minutes = Array.from({ length: Math.floor(60 / minuteStep) }, (_, i) => String(i * minuteStep).padStart(2,'0'))

  return (
    <div ref={ref} className="relative font-aumovio w-full">
      {label && <label className="block text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-1.5">{label}</label>}
      <button
        type="button" disabled={disabled}
        onClick={() => setOpen(o => !o)}
        className={`w-full flex items-center gap-2 px-3 py-2 rounded-xl border text-sm text-left
          bg-white dark:bg-[#1a1030] text-black/80 dark:text-white/80 transition-all duration-200
          ${open ? 'border-orange-400 ring-2 ring-orange-400/30' : 'border-grey-300 dark:border-grey-700'}
          ${disabled ? 'opacity-50 cursor-not-allowed' : ''}`}>
        <ClockIcon className="w-4 h-4 text-grey-400" />
        <span className={value ? '' : 'text-grey-400'}>
          {value || 'Select time'}
        </span>
      </button>
      {open && (
        <div className="absolute top-full left-0 mt-2 z-50 bg-white dark:bg-[#1a1030]
          border border-grey-200 dark:border-grey-700 rounded-xl shadow-2xl overflow-hidden animate-scale-in">
          <div className="flex">
            {/* Hours */}
            <div className="flex flex-col h-48 overflow-y-auto border-r border-grey-100 dark:border-grey-700 w-16">
              {hours.map(hv => (
                <button key={hv} onClick={() => emit(hv, m, ampm)}
                  className={`px-3 py-2 text-sm text-center hover:bg-orange-50 dark:hover:bg-orange-400/10
                    hover:text-orange-400 transition-colors
                    ${hv === h ? 'bg-orange-400 text-white font-aumovio-bold' : 'text-black/70 dark:text-white/70'}`}>
                  {String(hv).padStart(2,'0')}
                </button>
              ))}
            </div>
            {/* Minutes */}
            <div className="flex flex-col h-48 overflow-y-auto w-16">
              {minutes.map(mv => (
                <button key={mv} onClick={() => emit(h, mv, ampm)}
                  className={`px-3 py-2 text-sm text-center hover:bg-orange-50 dark:hover:bg-orange-400/10
                    hover:text-orange-400 transition-colors
                    ${mv === m ? 'bg-orange-400 text-white font-aumovio-bold' : 'text-black/70 dark:text-white/70'}`}>
                  {mv}
                </button>
              ))}
            </div>
            {/* AM/PM */}
            {use12Hour && (
              <div className="flex flex-col h-48 justify-center border-l border-grey-100 dark:border-grey-700 w-14">
                {['AM','PM'].map(ap => (
                  <button key={ap} onClick={() => emit(h, m, ap)}
                    className={`px-3 py-3 text-sm font-aumovio-bold text-center hover:bg-orange-50
                      dark:hover:bg-orange-400/10 hover:text-orange-400 transition-colors
                      ${ap === ampm ? 'text-orange-400' : 'text-grey-500'}`}>
                    {ap}
                  </button>
                ))}
              </div>
            )}
          </div>
        </div>
      )}
      {error && <p className="mt-1.5 text-xs text-danger-400 font-aumovio-bold">{error}</p>}
    </div>
  )
}

export default Timepicker
```

---

### Checkbox

```jsx
// src/components/forms/Checkbox.jsx
/**
 * Checkbox — Styled checkbox input.
 *
 * Props:
 *   id, name, label, checked, onChange, disabled, indeterminate
 *   description — helper text beneath label
 *   variant — 'default'|'card'
 *   error
 */
import { useRef, useEffect } from 'react'
import { CheckIcon, MinusIcon } from '@heroicons/react/24/outline'

export function Checkbox({
  id, name, label, checked = false, onChange, disabled = false,
  indeterminate = false, description, variant = 'default', error,
}) {
  const ref = useRef(null)
  useEffect(() => { if (ref.current) ref.current.indeterminate = indeterminate }, [indeterminate])

  const inputEl = (
    <div className="relative shrink-0 w-4 h-4">
      <input ref={ref} type="checkbox" id={id ?? name} name={name}
        checked={checked} onChange={onChange} disabled={disabled}
        className="sr-only peer" />
      <div className={`w-4 h-4 rounded border-2 transition-all duration-150 flex items-center justify-center
        ${checked || indeterminate
          ? 'bg-orange-400 border-orange-400'
          : 'bg-white dark:bg-grey-800 border-grey-300 dark:border-grey-600 peer-hover:border-orange-400'}
        ${disabled ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'}
        ${error ? 'border-danger-400' : ''}`}>
        {checked && !indeterminate && <CheckIcon className="w-3 h-3 text-white" strokeWidth={3} />}
        {indeterminate && <MinusIcon className="w-3 h-3 text-white" strokeWidth={3} />}
      </div>
    </div>
  )

  if (variant === 'card') return (
    <label className={`flex items-start gap-3 p-4 rounded-xl border cursor-pointer
      transition-all duration-200 font-aumovio
      ${checked ? 'border-orange-400 bg-orange-50 dark:bg-orange-400/5' : 'border-grey-200 dark:border-grey-700 bg-white dark:bg-[#1a1030] hover:border-orange-300'}
      ${disabled ? 'opacity-50 cursor-not-allowed' : ''}`}>
      {inputEl}
      <div>
        <p className={`text-sm font-aumovio-bold ${checked ? 'text-orange-400' : 'text-black/85 dark:text-white/85'}`}>{label}</p>
        {description && <p className="text-xs text-grey-400 mt-0.5">{description}</p>}
      </div>
    </label>
  )

  return (
    <div className="font-aumovio">
      <label className={`flex items-start gap-2.5 cursor-pointer
        ${disabled ? 'opacity-50 cursor-not-allowed' : ''}`}>
        {inputEl}
        {label && (
          <div>
            <span className="text-sm text-black/80 dark:text-white/80 font-aumovio">{label}</span>
            {description && <p className="text-xs text-grey-400 mt-0.5">{description}</p>}
          </div>
        )}
      </label>
      {error && <p className="mt-1 text-xs text-danger-400 font-aumovio-bold">{error}</p>}
    </div>
  )
}

export default Checkbox
```

---

### Radio

```jsx
// src/components/forms/Radio.jsx
/**
 * Radio — Radio button group.
 *
 * Props:
 *   name, options: [{ value, label, description?, disabled? }]
 *   value, onChange, label, error, disabled
 *   variant — 'default'|'card'|'button-group'
 *   orientation — 'vertical'|'horizontal'
 */
export function Radio({
  name, options = [], value, onChange, label: groupLabel, error, disabled = false,
  variant = 'default', orientation = 'vertical',
}) {
  if (variant === 'button-group') return (
    <div className="font-aumovio">
      {groupLabel && <p className="text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-1.5">{groupLabel}</p>}
      <div className="inline-flex border border-grey-200 dark:border-grey-700 rounded-xl overflow-hidden divide-x divide-grey-200 dark:divide-grey-700">
        {options.map(opt => (
          <label key={opt.value}
            className={`px-4 py-2 text-sm font-aumovio-bold cursor-pointer transition-colors
              ${opt.disabled || disabled ? 'opacity-40 cursor-not-allowed' : ''}
              ${value === opt.value
                ? 'bg-orange-400 text-white'
                : 'bg-white dark:bg-[#1a1030] text-grey-600 dark:text-grey-300 hover:bg-orange-50 hover:text-orange-400'}`}>
            <input type="radio" name={name} value={opt.value} checked={value === opt.value}
              onChange={() => !opt.disabled && !disabled && onChange?.(opt.value)}
              disabled={opt.disabled || disabled} className="sr-only" />
            {opt.label}
          </label>
        ))}
      </div>
    </div>
  )

  return (
    <fieldset className="font-aumovio">
      {groupLabel && <legend className="text-xs font-aumovio-bold text-black/70 dark:text-white/70 mb-2">{groupLabel}</legend>}
      <div className={`flex gap-3 ${orientation === 'horizontal' ? 'flex-row flex-wrap' : 'flex-col'}`}>
        {options.map(opt => (
          <label key={opt.value}
            className={`flex items-start gap-2.5 cursor-pointer
              ${variant === 'card' ? `p-4 rounded-xl border transition-all duration-200
                ${value === opt.value ? 'border-orange-400 bg-orange-50 dark:bg-orange-400/5' : 'border-grey-200 dark:border-grey-700 hover:border-orange-300'}` : ''}
              ${opt.disabled || disabled ? 'opacity-50 cursor-not-allowed' : ''}`}>
            <div className="relative shrink-0 mt-0.5">
              <input type="radio" name={name} value={opt.value} checked={value === opt.value}
                onChange={() => !opt.disabled && !disabled && onChange?.(opt.value)}
                disabled={opt.disabled || disabled}
                className="sr-only" />
              <div className={`w-4 h-4 rounded-full border-2 flex items-center justify-center transition-all duration-150
                ${value === opt.value
                  ? 'border-orange-400 bg-orange-400'
                  : 'border-grey-300 dark:border-grey-600 hover:border-orange-400 bg-white dark:bg-grey-800'}`}>
                {value === opt.value && <div className="w-1.5 h-1.5 rounded-full bg-white" />}
              </div>
            </div>
            <div>
              <p className={`text-sm font-aumovio ${value === opt.value ? 'text-orange-400 font-aumovio-bold' : 'text-black/80 dark:text-white/80'}`}>
                {opt.label}
              </p>
              {opt.description && <p className="text-xs text-grey-400 mt-0.5">{opt.description}</p>}
            </div>
          </label>
        ))}
      </div>
      {error && <p className="mt-1.5 text-xs text-danger-400 font-aumovio-bold">{error}</p>}
    </fieldset>
  )
}

export default Radio
```

---

### Toggle

```jsx
// src/components/forms/Toggle.jsx
/**
 * Toggle — Accessible on/off switch.
 *
 * Props:
 *   checked, onChange, label, description, disabled
 *   size     — 'sm'|'md'|'lg'
 *   color    — 'orange'|'success'|'purple'|'danger'
 *   labelPosition — 'right'|'left'
 */
const SIZES = {
  sm: { track:'w-8 h-4',   thumb:'w-3 h-3',   translate:'translate-x-4' },
  md: { track:'w-11 h-6',  thumb:'w-4 h-4',   translate:'translate-x-5' },
  lg: { track:'w-14 h-7',  thumb:'w-5 h-5',   translate:'translate-x-7' },
}

const COLORS = {
  orange:  'bg-orange-400',
  success: 'bg-success-400',
  purple:  'bg-purple-400',
  danger:  'bg-danger-400',
}

export function Toggle({
  checked = false, onChange, label, description, disabled = false,
  size = 'md', color = 'orange', labelPosition = 'right',
}) {
  const sz = SIZES[size] ?? SIZES.md
  const col = COLORS[color] ?? COLORS.orange

  const track = (
    <button
      type="button" role="switch" aria-checked={checked}
      onClick={() => !disabled && onChange?.(!checked)}
      disabled={disabled}
      className={`relative inline-flex items-center rounded-full shrink-0
        transition-colors duration-200 ease-in-out focus-visible:outline-none
        focus-visible:ring-2 focus-visible:ring-orange-400/50 focus-visible:ring-offset-2
        ${sz.track}
        ${checked ? col : 'bg-grey-300 dark:bg-grey-600'}
        ${disabled ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'}`}>
      <span className={`${sz.thumb} inline-block bg-white rounded-full shadow-sm
        transform transition-transform duration-200 ease-in-out mx-0.5
        ${checked ? sz.translate : 'translate-x-0'}`} />
    </button>
  )

  if (!label) return track

  return (
    <label className={`flex items-center gap-3 cursor-pointer font-aumovio
      ${disabled ? 'opacity-50 cursor-not-allowed' : ''}`}>
      {labelPosition === 'left' && (
        <div>
          <span className="text-sm text-black/80 dark:text-white/80 font-aumovio">{label}</span>
          {description && <p className="text-xs text-grey-400">{description}</p>}
        </div>
      )}
      {track}
      {labelPosition === 'right' && (
        <div>
          <span className="text-sm text-black/80 dark:text-white/80 font-aumovio">{label}</span>
          {description && <p className="text-xs text-grey-400">{description}</p>}
        </div>
      )}
    </label>
  )
}

export default Toggle
```

---

### Range

```jsx
// src/components/forms/Range.jsx
/**
 * Range — Styled range slider.
 *
 * Props:
 *   value, onChange, min, max, step, label, disabled
 *   showValue — boolean
 *   showTicks — boolean
 *   color     — 'orange'|'purple'|'success'
 */
const COLORS = {
  orange:  '#FF4208',
  purple:  '#4827AF',
  success: '#32CB70',
}

export function Range({
  value = 0, onChange, min = 0, max = 100, step = 1,
  label, disabled = false, showValue = true, showTicks = false, color = 'orange',
}) {
  const pct = ((value - min) / (max - min)) * 100
  const col = COLORS[color] ?? COLORS.orange

  return (
    <div className="font-aumovio">
      {(label || showValue) && (
        <div className="flex justify-between mb-1.5">
          {label && <span className="text-xs font-aumovio-bold text-black/70 dark:text-white/70">{label}</span>}
          {showValue && <span className="text-xs font-aumovio-bold text-orange-400">{value}</span>}
        </div>
      )}
      <div className="relative">
        <input
          type="range" value={value} min={min} max={max} step={step}
          onChange={e => onChange?.(parseFloat(e.target.value))}
          disabled={disabled}
          className={`w-full h-2 rounded-full appearance-none cursor-pointer
            bg-grey-200 dark:bg-grey-700
            [&::-webkit-slider-thumb]:appearance-none
            [&::-webkit-slider-thumb]:w-4 [&::-webkit-slider-thumb]:h-4
            [&::-webkit-slider-thumb]:rounded-full [&::-webkit-slider-thumb]:shadow-md
            [&::-webkit-slider-thumb]:border-2 [&::-webkit-slider-thumb]:border-white
            [&::-webkit-slider-thumb]:cursor-pointer [&::-webkit-slider-thumb]:transition-transform
            [&::-webkit-slider-thumb]:hover:scale-125
            ${disabled ? 'opacity-50 cursor-not-allowed' : ''}`}
          style={{
            background: `linear-gradient(to right, ${col} ${pct}%, #DCDCDC ${pct}%)`,
          }}
        />
        {showTicks && (
          <div className="flex justify-between mt-1">
            {[min, Math.round((min+max)/2), max].map(v => (
              <span key={v} className="text-xs text-grey-400">{v}</span>
            ))}
          </div>
        )}
      </div>
    </div>
  )
}

export default Range
```

---

### Floating Label

```jsx
// src/components/forms/FloatingLabel.jsx
/**
 * FloatingLabel — Input with animated floating label.
 *
 * Props:
 *   label, name, type, value, onChange, error, disabled, required, size
 */
export function FloatingLabel({
  label, name, type = 'text', value = '', onChange,
  error, disabled = false, required = false, id,
}) {
  const inputId = id ?? name
  const filled = value !== '' && value !== null && value !== undefined

  return (
    <div className="relative font-aumovio">
      <input
        id={inputId} name={name} type={type} value={value}
        onChange={onChange} disabled={disabled} required={required}
        placeholder=" "
        className={`peer w-full px-3.5 pt-5 pb-2 text-sm rounded-xl border
          bg-white dark:bg-[#1a1030] text-black/85 dark:text-white/85
          focus:outline-none focus:ring-2 focus:shadow-md transition-all duration-200
          disabled:opacity-50 disabled:cursor-not-allowed placeholder-transparent
          ${error
            ? 'border-danger-400 focus:ring-danger-400/30'
            : 'border-grey-300 dark:border-grey-700 focus:ring-orange-400/30 focus:border-orange-400'
          }`}
      />
      <label
        htmlFor={inputId}
        className={`absolute left-3.5 transition-all duration-200 pointer-events-none font-aumovio
          peer-placeholder-shown:top-3.5 peer-placeholder-shown:text-sm peer-placeholder-shown:text-grey-400
          peer-focus:top-1.5 peer-focus:text-xs peer-focus:text-orange-400 peer-focus:font-aumovio-bold
          ${filled ? 'top-1.5 text-xs text-grey-500 dark:text-grey-400 font-aumovio-bold' : 'top-3.5 text-sm text-grey-400'}`}>
        {label}{required && <span className="text-danger-400 ml-0.5">*</span>}
      </label>
      {error && <p className="mt-1.5 text-xs text-danger-400 font-aumovio-bold">{error}</p>}
    </div>
  )
}

export default FloatingLabel
```

---

## 8. Typography

### Headings

```jsx
// src/components/ui/typography/Heading.jsx
/**
 * Heading — H1–H6 with consistent Aumovio scale.
 *
 * Props:
 *   as       — 'h1'|'h2'|'h3'|'h4'|'h5'|'h6'
 *   size     — overrides tag size
 *   gradient — boolean
 *   align    — 'left'|'center'|'right'
 *   children
 */
const SCALE = {
  h1: 'text-4xl md:text-5xl font-extrabold tracking-tight leading-tight',
  h2: 'text-3xl md:text-4xl font-extrabold tracking-tight leading-tight',
  h3: 'text-2xl md:text-3xl font-bold tracking-tight',
  h4: 'text-xl md:text-2xl font-bold',
  h5: 'text-lg md:text-xl font-bold',
  h6: 'text-base md:text-lg font-bold',
}

const ALIGNS = { left:'text-left', center:'text-center', right:'text-right' }

export function Heading({ as: Tag = 'h2', size, gradient = false, align = 'left', className = '', children }) {
  const sz = SCALE[size ?? Tag] ?? SCALE.h2
  return (
    <Tag className={`font-aumovio-bold text-black dark:text-white ${sz} ${ALIGNS[align]} ${className}
      ${gradient ? 'bg-gradient-to-r from-orange-400 to-purple-400 bg-clip-text text-transparent' : ''}`}>
      {children}
    </Tag>
  )
}

// Convenience exports
export const H1 = (p) => <Heading as="h1" {...p} />
export const H2 = (p) => <Heading as="h2" {...p} />
export const H3 = (p) => <Heading as="h3" {...p} />
export const H4 = (p) => <Heading as="h4" {...p} />
export const H5 = (p) => <Heading as="h5" {...p} />
export const H6 = (p) => <Heading as="h6" {...p} />

export default Heading
```

**Usage examples:**
```jsx
<H1>Main Page Title</H1>
<H2 gradient align="center">Feature Heading</H2>
<H3>Section Title</H3>
```

---

### Paragraphs

```jsx
// src/components/ui/typography/Paragraph.jsx
/**
 * Paragraph — Body text with size and color variants.
 *
 * size  — 'xs'|'sm'|'base'|'lg'|'xl'
 * color — 'default'|'muted'|'primary'|'inverse'
 * lead  — boolean (intro paragraph style)
 */
const SZ = { xs:'text-xs', sm:'text-sm', base:'text-base', lg:'text-lg', xl:'text-xl' }
const COL = {
  default: 'text-black/75 dark:text-white/75',
  muted:   'text-grey-500 dark:text-grey-400',
  primary: 'text-orange-500',
  inverse: 'text-white',
}

export function Paragraph({ size = 'base', color = 'default', lead = false, className = '', children }) {
  return (
    <p className={`font-aumovio leading-relaxed ${SZ[size]??SZ.base} ${COL[color]??COL.default}
      ${lead ? 'text-lg md:text-xl text-black/60 dark:text-white/60 leading-loose' : ''}
      ${className}`}>
      {children}
    </p>
  )
}

export default Paragraph
```

---

### Blockquote

```jsx
// src/components/ui/typography/Blockquote.jsx
/**
 * Blockquote — Pull-quote or citation.
 *
 * Props: cite, variant ('default'|'border'|'card'), children
 */
const V = {
  default: 'border-l-4 border-orange-400 pl-6 py-1',
  border:  'border-l-4 border-purple-400 pl-6 py-1',
  card:    'bg-orange-50 dark:bg-orange-400/5 border border-orange-400/20 rounded-xl p-6',
}

export function Blockquote({ cite, variant = 'default', children }) {
  return (
    <figure className={`my-6 font-aumovio ${V[variant]??V.default}`}>
      <blockquote className="text-lg md:text-xl italic text-black/75 dark:text-white/75 leading-relaxed">
        "{children}"
      </blockquote>
      {cite && (
        <figcaption className="mt-3 text-sm font-aumovio-bold text-orange-400">
          — {cite}
        </figcaption>
      )}
    </figure>
  )
}

export default Blockquote
```

---

### Images

```jsx
// src/components/ui/typography/Image.jsx
/**
 * Image — Responsive image with optional caption, aspect ratio, rounded corners.
 *
 * Props: src, alt, caption, aspect ('square'|'video'|'wide'|'auto'),
 *   rounded, shadow, hover, fullWidth
 */
const ASPECT = { square:'aspect-square', video:'aspect-video', wide:'aspect-[21/9]', auto:'' }
const ROUND  = { none:'', sm:'rounded-lg', md:'rounded-xl', lg:'rounded-2xl', full:'rounded-full' }

export function Image({
  src, alt = '', caption, aspect = 'auto', rounded = 'md',
  shadow = true, hover = false, fullWidth = false, className = '',
}) {
  return (
    <figure className={`font-aumovio ${fullWidth ? 'w-full' : 'inline-block'}`}>
      <div className={`overflow-hidden ${ROUND[rounded]??ROUND.md}
        ${shadow ? 'shadow-xl' : ''} ${ASPECT[aspect]??ASPECT.auto}`}>
        <img src={src} alt={alt} loading="lazy" decoding="async"
          className={`${ASPECT[aspect] ? 'w-full h-full object-cover' : 'max-w-full h-auto'}
            ${hover ? 'transition-transform duration-500 hover:scale-105' : ''}
            ${className}`} />
      </div>
      {caption && (
        <figcaption className="mt-2 text-xs text-grey-400 text-center">{caption}</figcaption>
      )}
    </figure>
  )
}

export default Image
```

---

### Lists

```jsx
// src/components/ui/typography/List.jsx
/**
 * List — Ordered, unordered, or description list.
 *
 * Props:
 *   items    — string[] | [{ term, description }] (for dl)
 *   variant  — 'ul'|'ol'|'dl'|'check'|'icon'
 *   icons    — icon[] (one per item for icon variant)
 *   iconColor — string (Tailwind text color)
 *   size     — 'sm'|'md'|'lg'
 */
import { CheckCircleIcon } from '@heroicons/react/24/solid'

export function List({ items = [], variant = 'ul', icons, iconColor = 'text-orange-400', size = 'md' }) {
  const SZ = { sm:'text-xs', md:'text-sm', lg:'text-base' }
  const base = `font-aumovio leading-relaxed ${SZ[size]??SZ.md} space-y-2`

  if (variant === 'dl') return (
    <dl className={`${base} grid grid-cols-3 gap-x-6`}>
      {items.map((item, i) => (
        <div key={i} className="col-span-3 grid grid-cols-3 gap-x-4 items-baseline">
          <dt className="font-aumovio-bold text-black/70 dark:text-white/70 col-span-1">{item.term}</dt>
          <dd className="text-grey-500 dark:text-grey-400 col-span-2">{item.description}</dd>
        </div>
      ))}
    </dl>
  )

  if (variant === 'check') return (
    <ul className={base}>
      {items.map((item, i) => (
        <li key={i} className="flex items-start gap-2">
          <CheckCircleIcon className={`w-5 h-5 shrink-0 mt-0.5 ${iconColor}`} />
          <span className="text-black/75 dark:text-white/75">{item}</span>
        </li>
      ))}
    </ul>
  )

  if (variant === 'icon') return (
    <ul className={base}>
      {items.map((item, i) => {
        const Icon = icons?.[i]
        return (
          <li key={i} className="flex items-start gap-2">
            {Icon && <Icon className={`w-4 h-4 shrink-0 mt-1 ${iconColor}`} />}
            <span className="text-black/75 dark:text-white/75">{item}</span>
          </li>
        )
      })}
    </ul>
  )

  const Tag = variant === 'ol' ? 'ol' : 'ul'
  return (
    <Tag className={`${base} ${variant === 'ol' ? 'list-decimal pl-5' : 'list-disc pl-5'}`}>
      {items.map((item, i) => (
        <li key={i} className="text-black/75 dark:text-white/75 pl-1">{item}</li>
      ))}
    </Tag>
  )
}

export default List
```

---

### Links

```jsx
// src/components/ui/typography/Link.jsx
/**
 * Link — Styled anchor / NavLink.
 *
 * Props:
 *   href, external, variant ('default'|'primary'|'muted'|'danger'), children
 *   underline — 'always'|'hover'|'none'
 *   icon     — boolean (external link icon)
 */
import { NavLink } from 'react-router-dom'
import { ArrowTopRightOnSquareIcon } from '@heroicons/react/16/solid'

const V = {
  default: 'text-orange-400 hover:text-orange-500',
  primary: 'text-orange-500 font-aumovio-bold hover:text-orange-600',
  muted:   'text-grey-500 hover:text-grey-700 dark:text-grey-400 dark:hover:text-grey-200',
  danger:  'text-danger-400 hover:text-danger-500',
}

const UL = {
  always: 'underline underline-offset-2',
  hover:  'no-underline hover:underline underline-offset-2',
  none:   'no-underline',
}

export function Link({ href = '#', external = false, variant = 'default', underline = 'hover', icon = false, children }) {
  const cls = `inline-flex items-center gap-1 font-aumovio transition-colors duration-150
    focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-orange-400/50 rounded
    ${V[variant]??V.default} ${UL[underline]??UL.hover}`

  if (external) return (
    <a href={href} target="_blank" rel="noopener noreferrer" className={cls}>
      {children}
      {icon && <ArrowTopRightOnSquareIcon className="w-3.5 h-3.5 shrink-0 opacity-70" />}
    </a>
  )

  return <NavLink to={href} className={cls}>{children}</NavLink>
}

export default Link
```

---

### Text Utilities

```jsx
// src/components/ui/typography/Text.jsx
/**
 * Text — Inline text span with semantic variants.
 *
 * Props:
 *   variant — 'bold'|'italic'|'code'|'mark'|'del'|'sub'|'sup'|'kbd'|'small'|'lead'
 *   color   — any Tailwind color string
 *   children
 */
export function Text({ variant = 'bold', color, className = '', children }) {
  const col = color ?? ''

  if (variant === 'code') return (
    <code className={`font-mono text-[0.875em] px-1.5 py-0.5 rounded
      bg-grey-100 dark:bg-grey-800 text-orange-500 dark:text-orange-300
      border border-grey-200 dark:border-grey-700 ${col} ${className}`}>
      {children}
    </code>
  )
  if (variant === 'mark') return (
    <mark className={`bg-warn-100 text-black px-0.5 rounded ${col} ${className}`}>{children}</mark>
  )
  if (variant === 'del') return <del className={`line-through text-grey-400 ${col} ${className}`}>{children}</del>
  if (variant === 'bold') return <strong className={`font-aumovio-bold ${col} ${className}`}>{children}</strong>
  if (variant === 'italic') return <em className={`italic ${col} ${className}`}>{children}</em>
  if (variant === 'small') return <small className={`text-xs text-grey-500 ${col} ${className}`}>{children}</small>
  if (variant === 'lead') return <span className={`text-lg text-black/60 dark:text-white/60 leading-loose ${col} ${className}`}>{children}</span>
  if (variant === 'sub') return <sub className={`text-xs ${col} ${className}`}>{children}</sub>
  if (variant === 'sup') return <sup className={`text-xs ${col} ${className}`}>{children}</sup>

  return <span className={`${col} ${className}`}>{children}</span>
}

export default Text
```

---

### HR

```jsx
// src/components/ui/typography/Divider.jsx
/**
 * Divider — Horizontal rule with optional label.
 *
 * Props:
 *   label   — center text
 *   variant — 'solid'|'dashed'|'gradient'|'dot'
 *   spacing — 'sm'|'md'|'lg'
 */
const SP = { sm:'my-4', md:'my-6', lg:'my-10' }

export function Divider({ label, variant = 'solid', spacing = 'md' }) {
  const line = {
    solid:    'border-grey-200 dark:border-grey-700',
    dashed:   'border-dashed border-grey-200 dark:border-grey-700',
    gradient: 'border-0',
  }

  if (label) return (
    <div className={`flex items-center gap-4 font-aumovio ${SP[spacing]??SP.md}`}>
      <div className={`flex-1 ${variant !== 'gradient' ? `border-t ${line[variant]}` : 'h-px bg-gradient-to-r from-transparent to-grey-200 dark:to-grey-700'}`} />
      <span className="text-xs font-aumovio-bold text-grey-400 uppercase tracking-wider whitespace-nowrap">
        {label}
      </span>
      <div className={`flex-1 ${variant !== 'gradient' ? `border-t ${line[variant]}` : 'h-px bg-gradient-to-l from-transparent to-grey-200 dark:to-grey-700'}`} />
    </div>
  )

  if (variant === 'gradient') return (
    <div className={`${SP[spacing]??SP.md} h-px bg-gradient-to-r from-transparent via-grey-300 dark:via-grey-600 to-transparent`} />
  )

  if (variant === 'dot') return (
    <div className={`${SP[spacing]??SP.md} flex items-center justify-center gap-1.5`}>
      {[0,1,2].map(i => <span key={i} className="w-1 h-1 rounded-full bg-grey-300 dark:bg-grey-600" />)}
    </div>
  )

  return <hr className={`${SP[spacing]??SP.md} ${line[variant]??line.solid} border-t`} />
}

export default Divider
```

---

## 9. Charts (ApexCharts)

### Setup

```bash
npm install apexcharts react-apexcharts
```

### Base Config

```js
// src/utils/chartDefaults.js
import { TOKENS } from './tokens'

export const chartBase = {
  chart: {
    fontFamily: "'Aumovio', ui-sans-serif, system-ui",
    toolbar: { show: false },
    animations: { enabled: true, easing: 'easeinout', speed: 600 },
    background: 'transparent',
  },
  dataLabels: { enabled: false },
  grid: {
    borderColor: '#DCDCDC',
    strokeDashArray: 4,
    xaxis: { lines: { show: false } },
    yaxis: { lines: { show: true } },
  },
  tooltip: {
    theme: 'light',
    style: { fontFamily: "'Aumovio', sans-serif", fontSize: '12px' },
    x: { show: true },
  },
  legend: {
    fontFamily: "'Aumovio', sans-serif",
    fontSize: '12px',
    fontWeight: 700,
  },
  stroke: { curve: 'smooth', width: 2.5 },
  colors: [TOKENS.primary, TOKENS.secondary, TOKENS.blue, TOKENS.turquoise, TOKENS.success],
}

export const darkChartBase = {
  ...chartBase,
  chart: { ...chartBase.chart, background: 'transparent' },
  grid: { ...chartBase.grid, borderColor: '#303030' },
  tooltip: { ...chartBase.tooltip, theme: 'dark' },
  theme: { mode: 'dark' },
}
```

### Line Chart

```jsx
// src/components/charts/LineChart.jsx
import ReactApexChart from 'react-apexcharts'
import { chartBase } from '../../utils/chartDefaults'
import { useTheme } from '../../contexts/theme/ThemeContext'

export function LineChart({ series = [], categories = [], height = 300, title, stacked = false }) {
  const { isDark } = useTheme()

  const options = {
    ...chartBase,
    ...(isDark ? { theme: { mode: 'dark' } } : {}),
    chart: { ...chartBase.chart, type: 'line', stacked },
    xaxis: { categories, labels: { style: { fontFamily: 'Aumovio' } } },
    yaxis: { labels: { style: { fontFamily: 'Aumovio' } } },
    title: title ? { text: title, style: { fontFamily: 'Aumovio', fontWeight: 700 } } : undefined,
    markers: { size: 4, hover: { size: 7 } },
  }

  return <ReactApexChart type="line" options={options} series={series} height={height} />
}
```

### Bar Chart

```jsx
// src/components/charts/BarChart.jsx
import ReactApexChart from 'react-apexcharts'
import { chartBase } from '../../utils/chartDefaults'
import { useTheme } from '../../contexts/theme/ThemeContext'

export function BarChart({
  series = [], categories = [], height = 300, horizontal = false,
  title, stacked = false, rounded = true,
}) {
  const { isDark } = useTheme()

  const options = {
    ...chartBase,
    ...(isDark ? { theme: { mode: 'dark' } } : {}),
    chart: { ...chartBase.chart, type: 'bar', stacked },
    plotOptions: {
      bar: {
        horizontal,
        borderRadius: rounded ? 6 : 0,
        columnWidth: '60%',
        dataLabels: { position: 'top' },
      }
    },
    xaxis: { categories, labels: { style: { fontFamily: 'Aumovio' } } },
    yaxis: { labels: { style: { fontFamily: 'Aumovio' } } },
    title: title ? { text: title, style: { fontFamily: 'Aumovio', fontWeight: 700 } } : undefined,
  }

  return <ReactApexChart type="bar" options={options} series={series} height={height} />
}
```

### Area Chart

```jsx
// src/components/charts/AreaChart.jsx
import ReactApexChart from 'react-apexcharts'
import { chartBase } from '../../utils/chartDefaults'
import { useTheme } from '../../contexts/theme/ThemeContext'

export function AreaChart({ series = [], categories = [], height = 300, title, gradient = true }) {
  const { isDark } = useTheme()

  const options = {
    ...chartBase,
    ...(isDark ? { theme: { mode: 'dark' } } : {}),
    chart: { ...chartBase.chart, type: 'area' },
    fill: gradient
      ? { type: 'gradient', gradient: { opacityFrom: 0.5, opacityTo: 0.05, shadeIntensity: 1 } }
      : { type: 'solid', opacity: 0.2 },
    xaxis: { categories, labels: { style: { fontFamily: 'Aumovio' } } },
    yaxis: { labels: { style: { fontFamily: 'Aumovio' } } },
    title: title ? { text: title, style: { fontFamily: 'Aumovio', fontWeight: 700 } } : undefined,
    markers: { size: 0, hover: { size: 6 } },
  }

  return <ReactApexChart type="area" options={options} series={series} height={height} />
}
```

### Pie / Donut Chart

```jsx
// src/components/charts/DonutChart.jsx
import ReactApexChart from 'react-apexcharts'
import { chartBase, TOKENS } from '../../utils/chartDefaults'
import { useTheme } from '../../contexts/theme/ThemeContext'

export function DonutChart({ series = [], labels = [], height = 300, title, donut = true }) {
  const { isDark } = useTheme()

  const options = {
    ...chartBase,
    ...(isDark ? { theme: { mode: 'dark' } } : {}),
    chart: { ...chartBase.chart, type: donut ? 'donut' : 'pie' },
    labels,
    plotOptions: donut ? {
      pie: { donut: { size: '65%', labels: {
        show: true,
        total: { show: true, fontFamily: 'Aumovio', fontWeight: 700 }
      }}}
    } : {},
    title: title ? { text: title, style: { fontFamily: 'Aumovio', fontWeight: 700 } } : undefined,
  }

  return <ReactApexChart type={donut ? 'donut' : 'pie'} options={options} series={series} height={height} />
}
```

### Radial Bar Chart

```jsx
// src/components/charts/RadialChart.jsx
import ReactApexChart from 'react-apexcharts'
import { chartBase } from '../../utils/chartDefaults'
import { useTheme } from '../../contexts/theme/ThemeContext'

export function RadialChart({ series = [], labels = [], height = 300, title }) {
  const { isDark } = useTheme()

  const options = {
    ...chartBase,
    ...(isDark ? { theme: { mode: 'dark' } } : {}),
    chart: { ...chartBase.chart, type: 'radialBar' },
    labels,
    plotOptions: {
      radialBar: {
        track: { background: '#F0F0F0', strokeWidth: '80%' },
        dataLabels: {
          name: { fontFamily: 'Aumovio', fontWeight: 700, fontSize: '13px' },
          value: { fontFamily: 'Aumovio', fontWeight: 700, fontSize: '20px' },
        },
        hollow: { size: '30%' },
      }
    },
    title: title ? { text: title, style: { fontFamily: 'Aumovio', fontWeight: 700 } } : undefined,
  }

  return <ReactApexChart type="radialBar" options={options} series={series} height={height} />
}
```

### Heatmap Chart

```jsx
// src/components/charts/HeatmapChart.jsx
import ReactApexChart from 'react-apexcharts'
import { chartBase } from '../../utils/chartDefaults'
import { useTheme } from '../../contexts/theme/ThemeContext'

export function HeatmapChart({ series = [], height = 300, title }) {
  const { isDark } = useTheme()

  const options = {
    ...chartBase,
    ...(isDark ? { theme: { mode: 'dark' } } : {}),
    chart: { ...chartBase.chart, type: 'heatmap' },
    plotOptions: {
      heatmap: {
        shadeIntensity: 0.6,
        radius: 6,
        colorScale: {
          ranges: [
            { from: 0,  to: 25,  color: '#FFF5F2', name: 'low'  },
            { from: 26, to: 50,  color: '#FFB7A1', name: 'med'  },
            { from: 51, to: 75,  color: '#FF693B', name: 'high' },
            { from: 76, to: 100, color: '#FF4208', name: 'peak' },
          ]
        }
      }
    },
    title: title ? { text: title, style: { fontFamily: 'Aumovio', fontWeight: 700 } } : undefined,
  }

  return <ReactApexChart type="heatmap" options={options} series={series} height={height} />
}
```

### Scatter Chart

```jsx
// src/components/charts/ScatterChart.jsx
import ReactApexChart from 'react-apexcharts'
import { chartBase } from '../../utils/chartDefaults'
import { useTheme } from '../../contexts/theme/ThemeContext'

export function ScatterChart({ series = [], height = 300, title, xLabel = '', yLabel = '' }) {
  const { isDark } = useTheme()

  const options = {
    ...chartBase,
    ...(isDark ? { theme: { mode: 'dark' } } : {}),
    chart: { ...chartBase.chart, type: 'scatter' },
    xaxis: { title: { text: xLabel, style: { fontFamily: 'Aumovio', fontWeight: 700 } } },
    yaxis: { title: { text: yLabel, style: { fontFamily: 'Aumovio', fontWeight: 700 } } },
    markers: { size: 6, hover: { sizeOffset: 3 } },
    title: title ? { text: title, style: { fontFamily: 'Aumovio', fontWeight: 700 } } : undefined,
  }

  return <ReactApexChart type="scatter" options={options} series={series} height={height} />
}
```

### Chart Usage Examples

```jsx
// Dashboard stat charts
<LineChart
  title="Monthly Revenue"
  categories={['Jan','Feb','Mar','Apr','May','Jun']}
  series={[{ name: 'Revenue', data: [30000,42000,38000,55000,48000,62000] }]}
  height={280}
/>

<BarChart
  title="Sales by Category"
  categories={['Electronics','Clothing','Food','Books']}
  series={[
    { name: '2024', data: [44,55,41,64] },
    { name: '2025', data: [53,32,33,52] },
  ]}
  stacked
  height={280}
/>

<DonutChart
  title="Traffic Sources"
  labels={['Organic','Direct','Social','Referral']}
  series={[42,28,18,12]}
  height={280}
/>
```

---

## Appendix: File Structure

```
src/
├── assets/
│   ├── fonts/           # AUMOVIOScreen-Regular.ttf, AUMOVIOScreen-Bold.ttf
│   ├── img/
│   ├── aumovio/         # logo variants
│   └── styles/
│       ├── index.css    # @theme tokens + @font-face + animations
│       └── pre-set-styles.jsx
├── components/
│   ├── charts/          # LineChart, BarChart, AreaChart, DonutChart, ...
│   ├── feedback/        # ErrorBoundary, LoadingSpinner
│   ├── forms/           # Input, Textarea, Select, Toggle, Range, ...
│   ├── layout/
│   │   ├── footer/
│   │   ├── loading/
│   │   ├── navbar/
│   │   └── sidebar/
│   ├── routing/         # ProtectedRoute
│   └── ui/
│       ├── typography/  # Heading, Paragraph, Text, List, Link, Divider, ...
│       ├── Accordion.jsx
│       ├── Alert.jsx
│       ├── Avatar.jsx
│       ├── Badge.jsx
│       ├── Banner.jsx
│       ├── BottomNav.jsx
│       ├── Breadcrumb.jsx
│       ├── Button.jsx
│       ├── ButtonGroup.jsx
│       ├── Card.jsx
│       ├── Carousel.jsx
│       ├── ChatBubble.jsx
│       ├── Clipboard.jsx
│       ├── Datepicker.jsx
│       ├── DeviceMockup.jsx
│       ├── Drawer.jsx
│       ├── Dropdown.jsx
│       ├── Gallery.jsx
│       ├── Indicator.jsx
│       ├── Jumbotron.jsx
│       ├── KBD.jsx
│       ├── ListGroup.jsx
│       ├── Modal.jsx
│       ├── Pagination.jsx
│       ├── Popover.jsx
│       ├── Progress.jsx
│       ├── QRCode.jsx
│       ├── Rating.jsx
│       ├── Skeleton.jsx
│       ├── SpeedDial.jsx
│       ├── Spinner.jsx
│       ├── Stepper.jsx
│       ├── Table.jsx
│       ├── Tabs.jsx
│       ├── ThemeToggle.jsx
│       ├── Timeline.jsx
│       ├── Tooltip.jsx
│       └── toast.utils.js
├── contexts/
│   ├── layout/          # LayoutContext
│   ├── security/        # CsrfContext
│   └── theme/           # ThemeContext
├── features/
│   ├── auth/
│   └── dashboard/
├── hooks/
│   ├── useDebounce.js
│   ├── useDocumentTitle.js
│   └── usePagination.js
├── middleware/
│   ├── authentication/  # AuthMiddleware
│   ├── security/        # CsrfMiddleware
│   └── HttpClient.js
├── utils/
│   ├── chartDefaults.js
│   ├── formatters.js
│   ├── storage.js
│   ├── tokens.js
│   └── validators.js
└── views/
    └── errors/          # ClientErrorResponses
```

---

*Last updated: Aumovio Design System v2.0 — React 19 + Tailwind v4 + ApexCharts*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jm-Paunlagui) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
