## resumemindai-ui

> **Stack:** Next.js 14+ (App Router) | TypeScript 5.x | Tailwind CSS v4 | React 19.x | Dark Purple Theme

# ResumeMindAI-UI - Windsurf AI Coding Rules

## 🎯 Project Context

**Stack:** Next.js 14+ (App Router) | TypeScript 5.x | Tailwind CSS v4 | React 19.x | Dark Purple Theme

**Purpose:** Landing page with AI-powered career intelligence features

**Architecture:** Component-based, feature-driven folders, utility-first Tailwind CSS

---

## 🎨 CRITICAL: Color Palette (DO NOT MODIFY)

### Core CSS Variables (app/globals.css)

**NEVER change these values:**

```css
--background-dark: #0f172a    /* Main page background */
--surface-dark: #1e293b       /* Cards, surfaces */
--primary: #8b5cf6            /* Purple - primary brand color */
--primary-glow: rgba(139, 92, 246, 0.4)  /* Purple glow */
--accent-blue: #3b82f6        /* Secondary blue */
```

### Complete Color Catalog & Usage

#### Purple (Primary Brand)
- **Hex:** #8b5cf6
- **Tailwind:** `bg-primary`, `text-primary`, `border-primary`
- **Usage:** CTA buttons, logos, accent text, status badges
- **Hover state:** `violet-600` (#7c3aed)
- **Opacity variants:** `bg-primary/10`, `bg-primary/5`, `bg-primary/80`

**Examples:**
```tsx
// Logo background
<div className="bg-primary rounded-lg shadow-lg shadow-primary/20">

// Primary CTA
<button className="bg-primary hover:bg-violet-600 text-white shadow-primary/30">

// Status badge
<div className="bg-primary/5 border border-primary/20 text-primary">

// Section heading
<h2 className="text-primary font-bold">
```

#### Emerald (Success/Live)
- **Hex:** #34d399 (emerald-400), #6ee7b7 (emerald-300)
- **Usage:** Live badges, success states, positive actions
- **Examples:**
```tsx
// Live indicator dot
<span className="bg-emerald-400 animate-ping">

// CTA accent border
<button className="border-emerald-300/70">
```

#### Blue (Interactive)
- **Hex:** #60a5fa (blue-400), #3b82f6 (blue-500)
- **Usage:** Interactive icons, secondary features
- **Examples:**
```tsx
// Feature icon
<div className="bg-blue-500/10">
  <span className="text-blue-400">hub</span>
</div>
```

#### Orange (Warning/Highlight)
- **Hex:** #fb923c (orange-400), #f97316 (orange-500)
- **Usage:** Warning states, insight icons
- **Examples:**
```tsx
// Feature icon
<div className="bg-orange-500/10">
  <span className="text-orange-400">insights</span>
</div>
```

#### Slate Grays (Text Hierarchy)
- **slate-400** (#94a3b8) - Body text, descriptions
- **slate-700** (#334155) - Borders, dividers
- **slate-800** (#1e293b) - Card hovers, dark surfaces
- **Examples:**
```tsx
<p className="text-slate-400">Description text</p>
<div className="border-slate-700/50">
<div className="hover:bg-slate-800">
```

#### Text Colors
- `text-white` - Headings, prominent content, navigation
- `text-slate-400` - Body text, descriptions
- `text-primary` - Purple accent text
- `text-blue-400`, `text-emerald-400`, `text-orange-400` - Feature icons

#### Border Colors
- `border-primary/20` - Purple subtle borders
- `border-slate-800/50`, `border-slate-700/50` - Dark borders
- `border-emerald-300/70` - Green accent borders
- `border-white/10` - Glass effect borders

#### Shadow Colors
- `shadow-primary/30`, `shadow-primary/25`, `shadow-primary/20` - Purple glows
- `shadow-xl` - Standard shadow with purple tint

### Custom CSS Classes (Preserve)

```css
/* Purple to indigo gradient text */
.text-gradient {
  background: linear-gradient(135deg, #c084fc 0%, #6366f1 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* Purple glow shadow */
.dashboard-mockup-shadow {
  box-shadow: 0 0 50px -12px var(--primary-glow);
}

/* Glassmorphism effect */
.glass-card {
  background: rgba(30, 41, 59, 0.7);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.1);
}
```

---

## 🎨 Color Preservation Guide

### Decision Tree for New Elements

When adding new UI elements, ask:

1. **Primary action/CTA?** → `bg-primary` + `hover:bg-violet-600`
2. **Success/live indicator?** → `emerald-400` or `emerald-300`
3. **Interactive icon?** → `blue-400` or `blue-500`
4. **Warning/highlight?** → `orange-400` or `orange-500`
5. **Body text?** → `text-slate-400`
6. **Heading?** → `text-white` or `text-primary`
7. **Card/panel?** → `glass-card` class
8. **Border?** → `border-slate-700/50` or `border-white/10`

### ✅ CORRECT: Adding a Feature Card

```tsx
export default function NewFeatureCard() {
  return (
    <div className="glass-card p-6 rounded-xl hover:scale-105 transition-transform">
      <div className="w-12 h-12 bg-emerald-500/10 rounded-lg flex items-center justify-center mb-4">
        <span className="material-symbols-outlined text-2xl text-emerald-400">
          check_circle
        </span>
      </div>
      <h3 className="text-xl font-semibold text-white mb-2">New Feature</h3>
      <p className="text-slate-400">Feature description goes here</p>
    </div>
  );
}
```

**Why this is correct:**
- Uses `glass-card` class for consistency
- Opacity-based icon background (`bg-emerald-500/10`)
- Proper text color hierarchy (`text-white`, `text-slate-400`)
- Material Symbols icon
- Matching Tailwind color classes

### ❌ WRONG: Common Mistakes

```tsx
// ❌ DON'T DO THIS
export default function BadFeatureCard() {
  return (
    <div className="bg-gray-900 p-6" style={{backgroundColor: '#1a1a1a'}}>
      <div className="bg-green-200" style={{width: '48px'}}>
        <span style={{color: '#00ff00'}}>✓</span>
      </div>
      <h3 style={{color: '#ffffff', fontSize: '20px'}}>Feature</h3>
      <p className="text-gray-500">Description</p>
    </div>
  );
}
```

**Why this is wrong:**
- ❌ `bg-gray-900` instead of `glass-card`
- ❌ Inline styles with hardcoded colors
- ❌ Wrong green shade (`green-200` instead of `emerald-400`)
- ❌ Emoji instead of Material Symbols icon
- ❌ `text-gray-500` instead of `text-slate-400`

### Common Color Combinations

**Primary CTA Button:**
```tsx
className="bg-primary hover:bg-violet-600 text-white px-6 py-3 rounded-lg
           font-semibold shadow-lg shadow-primary/30 transition-colors"
```

**Secondary Glass Button:**
```tsx
className="glass-card text-white px-6 py-3 rounded-lg font-semibold
           hover:bg-slate-800 transition-all border border-slate-700/50"
```

**Feature Icon Container:**
```tsx
// Purple variant
className="w-12 h-12 bg-primary/10 rounded-lg flex items-center justify-center"
// Icon: className="text-primary"

// Blue variant
className="w-12 h-12 bg-blue-500/10 rounded-lg flex items-center justify-center"
// Icon: className="text-blue-400"

// Emerald variant
className="w-12 h-12 bg-emerald-500/10 rounded-lg flex items-center justify-center"
// Icon: className="text-emerald-400"
```

**Status Badge:**
```tsx
className="inline-flex items-center px-4 py-1.5 rounded-full
           border border-primary/20 bg-primary/5 text-primary"
```

---

## 📁 Component Structure (MANDATORY)

### Folder Organization

```
app/components/
  ├── layout/      # Navbar, Footer
  ├── hero/        # Hero sections, banners
  ├── features/    # Feature cards, grids
  ├── process/     # Process steps, timelines
  ├── trust/       # Trust badges, tech banners
  └── cta/         # Call-to-action sections
```

**Rules:**
- **Feature-based folders** (NOT `/buttons`, `/cards`, `/common`)
- **PascalCase files:** `HeroSection.tsx`, `FeatureCard.tsx`
- **kebab-case folders:** `call-to-action/`, `pricing-section/`
- One component per file
- Default exports

---

## ✅ Component Patterns (MUST FOLLOW)

### 1. Data-Driven Mapping

✅ **PREFERRED PATTERN:**

```tsx
// FeaturesSection.tsx
import FeatureCard from './FeatureCard';

export default function FeaturesSection() {
  const features = [
    {
      icon: 'auto_awesome',
      iconColor: 'bg-primary/10',
      iconTextColor: 'text-primary',
      title: 'AI-Powered Analysis',
      description: 'Advanced AI extraction...'
    },
    {
      icon: 'hub',
      iconColor: 'bg-blue-500/10',
      iconTextColor: 'text-blue-400',
      title: 'Interactive Explorer',
      description: 'Navigate your career...'
    }
  ];

  return (
    <section className="py-24">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
          {features.map((feature, index) => (
            <FeatureCard key={index} {...feature} />
          ))}
        </div>
      </div>
    </section>
  );
}
```

### 2. Typed Props (Required)

```tsx
// FeatureCard.tsx
interface FeatureCardProps {
  icon: string;
  iconColor: string;
  iconTextColor: string;
  title: string;
  description: string;
}

export default function FeatureCard({
  icon,
  iconColor,
  iconTextColor,
  title,
  description
}: FeatureCardProps) {
  return (
    <div className="glass-card p-6 rounded-xl group hover:scale-105 transition-transform">
      <div className={`w-12 h-12 rounded-lg ${iconColor} flex items-center justify-center mb-4`}>
        <span className={`material-symbols-outlined text-2xl ${iconTextColor}`}>
          {icon}
        </span>
      </div>
      <h3 className="text-xl font-semibold text-white mb-2">{title}</h3>
      <p className="text-slate-400">{description}</p>
    </div>
  );
}
```

### 3. Composition Pattern

✅ **CORRECT:** Break large components into sub-components (< 200 lines)

```tsx
// hero/HeroSection.tsx
import DashboardMockup from './DashboardMockup';

export default function HeroSection() {
  return (
    <header className="relative pt-16 pb-32">
      {/* Hero content */}
      <DashboardMockup />
    </header>
  );
}
```

❌ **WRONG:** 500-line monolithic component

### 4. NO Hardcoding in Pages

❌ **ANTI-PATTERN:**
```tsx
// app/page.tsx - DON'T
export default function Home() {
  return (
    <section>
      <h1>Welcome</h1>
      <button>Get Started</button>
      {/* 100 lines of JSX */}
    </section>
  );
}
```

✅ **CORRECT:**
```tsx
// app/page.tsx
import HeroSection from './components/hero/HeroSection';

export default function Home() {
  return <HeroSection />;
}

// app/components/hero/HeroSection.tsx
export default function HeroSection() {
  return (
    <section>
      <h1>Welcome</h1>
      <button>Get Started</button>
    </section>
  );
}
```

---

## 🛠️ TypeScript Standards

### Strict Typing (Required)

✅ **DO:**
```tsx
interface ButtonProps {
  variant: 'primary' | 'secondary';
  children: React.ReactNode;
  onClick?: () => void;
}

function Button({ variant, children, onClick }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}
```

❌ **DON'T:**
```tsx
function Button(props: any) {  // ❌ Never use 'any'
  return <button>{props.children}</button>;
}
```

### Path Aliases

```tsx
// ✅ Use @/ for imports
import { FeatureCard } from '@/app/components/features/FeatureCard';

// ❌ Avoid long relative paths
import { FeatureCard } from '../../../components/features/FeatureCard';
```

---

## 🎨 Tailwind CSS Rules

### Class Organization Order

```tsx
className="
  flex items-center justify-center    // Layout
  gap-4 p-6 mb-8                      // Spacing
  text-lg font-semibold               // Typography
  text-white bg-primary               // Colors
  rounded-lg shadow-lg                // Effects
  hover:scale-105 transition-transform // Interactions
"
```

### Responsive Design (Mandatory)

```tsx
// ✅ Always add breakpoints
className="text-4xl lg:text-5xl"
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6"
className="py-20 lg:py-32"
className="hidden md:flex"

// ❌ Don't skip mobile
className="text-5xl"  // Too large on mobile
className="grid-cols-4"  // Breaks on small screens
```

### Component-Specific Classes

**Buttons:**
```tsx
// Primary CTA
"bg-primary hover:bg-violet-600 text-white px-6 py-3 rounded-lg font-semibold shadow-lg shadow-primary/30 transition-colors"

// Secondary
"glass-card text-white px-6 py-3 rounded-lg font-semibold hover:bg-slate-800 transition-all border border-slate-700/50"
```

**Cards:**
```tsx
"glass-card p-6 rounded-xl group hover:scale-105 transition-transform"
```

**Sections:**
```tsx
"py-20 lg:py-32"
"max-w-7xl mx-auto px-4 sm:px-6 lg:px-8"
```

### Layout & Spacing Standards

- **Container max:** `max-w-7xl mx-auto`
- **Container padding:** `px-4 sm:px-6 lg:px-8`
- **Section spacing:** `py-20 lg:py-32`
- **Card padding:** `p-6` or `p-8`
- **Grid gaps:** `gap-6` (small), `gap-8` (medium), `gap-12` (large)

---

## 🎭 Icons & Assets

### Material Symbols Only

✅ **CORRECT:**
```tsx
<span className="material-symbols-outlined text-2xl text-primary">
  auto_awesome
</span>
```

❌ **WRONG - Don't install:**
```tsx
import { SparklesIcon } from '@heroicons/react/24/outline';  // ❌
import { FaStar } from 'react-icons/fa';  // ❌
```

**Common icons:**
- `auto_awesome` - AI/magic
- `hub` - Graph/network
- `neurology` - Intelligence
- `insights` - Analytics
- `play_circle` - Video

**Reference:** [Google Material Symbols](https://fonts.google.com/icons)

### Images

```tsx
// ✅ Use Next.js Image
import Image from 'next/image';

<Image src="/logo.png" alt="Logo" width={200} height={200} />

// ❌ Don't use img
<img src="/logo.png" />  // Missing optimization
```

---

## ✅ Testing

- Use Vitest; add/refresh tests for any new or modified components.
- Prefer colocated __tests__ folders.
- Run `npm run test:coverage` before merging.

---

## 🚫 Prohibited Actions

**DO NOT:**

1. ❌ Change color variables in `globals.css`
2. ❌ Install UI libraries (Material-UI, Ant Design, Chakra)
3. ❌ Install icon libraries (use Material Symbols)
4. ❌ Use CSS modules or styled-components (Tailwind only)
5. ❌ Create class components (functional only)
6. ❌ Hardcode JSX in `page.tsx` (extract to components)
7. ❌ Skip TypeScript types
8. ❌ Use `any` type
9. ❌ Create `/common` or `/shared` folders
10. ❌ Add inline styles when Tailwind exists
11. ❌ Remove hover/focus states
12. ❌ Skip responsive breakpoints
13. ❌ Use pixel values (use Tailwind spacing)
14. ❌ Add new fonts (use Geist Sans/Mono/Inter)
15. ❌ Use different purple shades (stick to `--primary`)

---

## ✅ Pre-Commit Checklist

- [ ] Component in correct feature folder
- [ ] TypeScript interfaces for all props
- [ ] Uses CSS variables from color catalog
- [ ] Tailwind classes (no custom CSS unless necessary)
- [ ] Responsive breakpoints (`md:`, `lg:`)
- [ ] Material Symbols icons (if needed)
- [ ] No hardcoded JSX in page files
- [ ] Hover/focus states present
- [ ] No TypeScript/ESLint errors
- [ ] Colors match palette (purple, emerald, blue, orange, slate)
- [ ] Glass-card class for cards
- [ ] Consistent spacing (`py-20`, `gap-8`)
- [ ] Vitest tests added/updated for any new or changed components (prefer colocated __tests__, run `npm run test:coverage`)
- [ ] ESLint passes (run `npm run lint:fix`)
- [ ] Prettier passes (run `npm run format`)

---

## 📚 Reference Components

**Study these for patterns:**

- [app/components/features/FeaturesSection.tsx](resume-mind-ai/app/components/features/FeaturesSection.tsx) - Data mapping
- [app/components/features/FeatureCard.tsx](resume-mind-ai/app/components/features/FeatureCard.tsx) - Typed props, reusable card
- [app/components/process/HowItWorksSection.tsx](resume-mind-ai/app/components/process/HowItWorksSection.tsx) - Steps pattern
- [app/components/hero/HeroSection.tsx](resume-mind-ai/app/components/hero/HeroSection.tsx) - Composition, colors
- [app/components/layout/Navbar.tsx](resume-mind-ai/app/components/layout/Navbar.tsx) - Sticky nav, responsive

---

## 🎯 Quick Reference

### Common Patterns

```tsx
// Section wrapper
<section className="py-20 lg:py-32">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    {/* Content */}
  </div>
</section>

// Glass card
<div className="glass-card p-6 rounded-xl hover:scale-105 transition-transform">

// Primary CTA
<button className="bg-primary hover:bg-violet-600 text-white px-6 py-3 rounded-lg font-semibold shadow-lg shadow-primary/30">

// Gradient text
<span className="text-gradient">Knowledge Graph</span>

// Responsive grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">

// Feature icon
<div className="w-12 h-12 bg-primary/10 rounded-lg flex items-center justify-center">
  <span className="material-symbols-outlined text-primary">auto_awesome</span>
</div>
```

---

## 🔥 Golden Rules

1. **Purple theme** - Don't change colors
2. **Component extraction** - No hardcoding in pages
3. **Tailwind-first** - Utility classes over custom CSS
4. **Type everything** - Strict TypeScript, no `any`
5. **Feature folders** - Domain-based organization
6. **Data + map** - Array patterns for repeated UI
7. **Mobile-first** - Always responsive
8. **Material Symbols** - Only icon system
9. **Glass aesthetic** - Use `glass-card` class
10. **Consistent spacing** - Follow existing patterns

---

## 📋 File Structure Template

```tsx
// 1. Imports
import React from 'react';
import Link from 'next/link';

// 2. Types
interface ComponentProps {
  title: string;
  description: string;
}

// 3. Data (if needed)
const items = [
  { id: 1, name: 'Item 1' },
];

// 4. Component
export default function ComponentName({ title, description }: ComponentProps) {
  // Hooks
  const [state, setState] = React.useState<string>('');

  // Handlers
  const handleClick = () => {
    // ...
  };

  // JSX
  return (
    <section className="py-20 lg:py-32">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <h2 className="text-4xl font-bold text-white">{title}</h2>
        <p className="text-slate-400">{description}</p>
      </div>
    </section>
  );
}
```

---

## 🔧 Commands

```bash
npm run dev     # Development server
npm run build   # Production build
npm start       # Production server
npm run lint    # ESLint
npm run lint:fix  # ESLint fix
npm run format  # Prettier
npm run test    # Vitest
npm run test:coverage  # Vitest with coverage
```

---

## 📚 Key Dependencies

- Next.js 16.1.1 (App Router)
- React 19.2.3
- TypeScript 5.x
- Tailwind CSS 4.x
- Fonts: Geist Sans, Geist Mono, Inter
- Icons: Material Symbols Outlined (CDN)

---

**When in doubt:** Reference existing components in `app/components/` for approved patterns. Consistency is key!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arivuforge) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
