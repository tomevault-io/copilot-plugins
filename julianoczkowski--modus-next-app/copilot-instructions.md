## modus-next-app

> This is a Next.js 15 application demonstrating Trimble's Modus Web Components design system integration with modern React patterns. It features React 19, TypeScript support, comprehensive component examples, and strict design system compliance enforced by automated linting.

# Project Overview

This is a Next.js 15 application demonstrating Trimble's Modus Web Components design system integration with modern React patterns. It features React 19, TypeScript support, comprehensive component examples, and strict design system compliance enforced by automated linting.

**Main Technologies:**

- Next.js 15 (App Router + Turbopack)
- React 19
- TypeScript 5
- Modus Web Components React package
- Tailwind CSS 4
- Modus Icons (Field Systems)

**Architecture:**

The project follows the Next.js App Router architecture with server-side rendering and client-side interactivity. The main application code is located in the `app` directory. Reusable React components are in `app/components`, and global styles are in `app/globals.css`. The `ThemeContext` in `app/contexts/ThemeContext.tsx` manages 6-theme system switching (4 standard Modus themes + 2 Trimble Connect themes).

## 🚨 CRITICAL: Development Workflow & Testing

### MANDATORY: Run Linting Before Changes

```bash
# 🔍 Run all linting checks before any code changes
npm run lint:styles && npm run lint:colors && npm run lint:icons && npm run lint:semantic
```

### Chrome DevTools Testing (Use MCP)

**ALWAYS use Chrome DevTools MCP for testing:**

```bash
# Start dev server first
npm run dev
```

**Then test with MCP:**

- Navigate to `http://localhost:3000`
- Check console for JavaScript errors
- Test all interactive elements (buttons, forms)
- Verify responsive design (mobile/desktop)
- Test theme switching if applicable
- Run accessibility checks

## Essential Development Commands

### Core Development

```bash
npm install              # Install dependencies
npm run dev             # Start development server (Turbopack enabled)
npm run build           # Production build (Turbopack enabled)
npm start               # Start production server
```

### Quality Assurance

```bash
npm run lint            # Run ESLint
npm run type-check      # TypeScript compilation check
npm run lint:colors     # Verify Modus color compliance (CRITICAL)
npm run lint:icons      # Verify Modus icons
npm run lint:semantic   # Verify if app is using semantic HTML
npm run lint:styles     # Verify Modus styles
npm run lint:borders    # Verify border usage is correct
```

### Git Workflow

- Pre-commit hooks automatically run `lint:colors` on staged files
- All color violations must be resolved before commit
- Husky + lint-staged configuration enforces compliance

## Critical Architecture Patterns

### 1. Modus Web Components Integration

**MANDATORY:** Use React 19-compatible package (`@trimble-oss/moduswebcomponents-react@^1.0.0-react19`)

**Setup Pattern:**

```tsx
// app/components/ModusProvider.tsx (Client Component)
"use client";
import "@trimble-oss/moduswebcomponents-react/modus-wc-styles.css";

// Component Usage
import { ModusWcButton } from "@trimble-oss/moduswebcomponents-react";
```

**FORBIDDEN:**

- Manual `defineCustomElements()` calls
- Raw web component tags (`<modus-wc-button>`)
- Direct loader imports

### 2. Event Handling (CRITICAL PATTERN)

**Problem:** React event props don't work reliably with Modus Web Components.

**MANDATORY Solution:**

```tsx
"use client";
import { useRef, useEffect } from "react";

const componentRef = useRef<any>(null);

useEffect(() => {
  const component = componentRef.current;
  if (component) {
    const handleEvent = (event: CustomEvent) => {
      // Use componentRef.current, NEVER event.target
      component.someProperty = newValue;
    };

    component.addEventListener("eventName", handleEvent);
    return () => component.removeEventListener("eventName", handleEvent);
  }
}, []);

return <ModusWcComponent ref={componentRef} />;
```

### 3. Design System Colors (ENFORCED BY LINTING)

**🎯 CRITICAL: Use Design System Tailwind Classes from globals.css**

**ONLY ALLOWED:** Design system Tailwind classes (mapped from Modus CSS variables in `globals.css`)

```tsx
// ✅ CORRECT - Use design system Tailwind classes
<div className="bg-background text-foreground">
<div className="bg-card text-card-foreground">
<div className="bg-primary text-primary-foreground">
<div className="bg-success text-success-foreground">
<div className="bg-destructive text-destructive-foreground">
<div className="bg-warning text-warning-foreground">
<div className="bg-muted text-muted-foreground">
<div className="bg-secondary text-secondary-foreground">
```

**Design System Mapping (from globals.css):**

The design system maps Modus CSS variables to Tailwind classes:

```css
/* Base Colors (theme-adaptive) */
--background: var(--modus-wc-color-base-page); /* bg-background */
--foreground: var(--modus-wc-color-base-content); /* text-foreground */
--card: var(--modus-wc-color-base-100); /* bg-card */
--border: var(--modus-wc-color-base-200); /* border-border */
--muted: var(--modus-wc-color-base-200); /* bg-muted */
--secondary: var(--modus-wc-color-base-300); /* bg-secondary */

/* Semantic Colors (theme-consistent) */
--primary: var(--modus-wc-color-info); /* bg-primary */
--destructive: var(--modus-wc-color-error); /* bg-destructive */
--warning: var(--modus-wc-color-warning); /* bg-warning */
--success: var(--modus-wc-color-success); /* bg-success */
```

**Usage Examples:**

```tsx
// ✅ CORRECT - Use design system Tailwind classes
<div className="bg-background text-foreground border-default">
<div className="bg-primary text-primary-foreground">
<div className="bg-card text-card-foreground">
<div className="bg-muted text-muted-foreground">

// ❌ FORBIDDEN (Will fail lint)
<div style={{ backgroundColor: "var(--modus-wc-color-base-page)" }}>
<div style={{ color: "var(--modus-wc-color-info)" }}>
<div style={{ backgroundColor: "#ffffff" }}>
<div className="bg-blue-500 text-red-400">
```

### 4. Component Architecture

**MANDATORY:** Single configurable component pattern

```tsx
// ✅ CORRECT: One flexible component
interface ModusButtonProps {
  color?: "primary" | "secondary" | "tertiary" | "warning" | "danger";
  variant?: "filled" | "outlined" | "borderless";
  size?: "xs" | "sm" | "md" | "lg";
  // All configuration options
}

// ❌ FORBIDDEN: Multiple specific components
// ModusButtonPrimary.tsx
// ModusButtonSecondary.tsx
```

### 5. Styling Standards

**Preferred Approach:** Tailwind utility classes with design system colors

```tsx
<div className="max-w-5xl mx-auto p-8 bg-card rounded-lg border-default">
```

**Critical Border Rule:**

```tsx
// ❌ WRONG - Tailwind border classes don't work in v4
<div className="border border-border">

// ✅ CORRECT - Use our border utility classes with design system colors
<div className="border-default">
<div className="border-dashed">
```

**Avoid:**

- Inline styles (except dynamic values)
- CSS modules
- Semantic HTML elements (`<h1>`, `<section>`) - use `<div>` with Tailwind

**Typography:**

```tsx
// ✅ CORRECT
<div className="text-4xl font-semibold text-foreground">Title</div>

// ❌ WRONG (Browser defaults interfere)
<h1 className="text-4xl font-semibold">Title</h1>
```

## 🎯 Key Rules Summary

### Essential Development Rules

1. **🚨 ALWAYS run linting commands before making changes**
2. **🧪 Use Chrome DevTools MCP for testing implementations**
3. **📋 Create implementation guides for major features**
4. **🎨 Use border utility classes (not Tailwind border classes)**
5. **📝 Use div elements (not semantic HTML) for consistent Tailwind styling**
6. **🎛️ Let Modus components handle their own state (don't control from React)**
7. **🔧 Use ref-based event handling for Modus Web Components**

### Final Quality Checklist

- [ ] ✅ All 4 linting commands pass (0 violations)
- [ ] ✅ Chrome DevTools shows no console errors
- [ ] ✅ All interactive elements work correctly
- [ ] ✅ Responsive design tested
- [ ] ✅ Theme compatibility verified (if themes present)
- [ ] ✅ Connect themes tested (if building Trimble Connect Web Applications)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julianoczkowski)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/julianoczkowski)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
