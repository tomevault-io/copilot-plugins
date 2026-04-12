## catfood

> This document describes the **Cat Food Reviews** project architecture, code style preferences, and context to use in future sessions. Code style was inferred from the codebase; edit this file to reflect your own preferences.

# Project Context for AI Sessions

This document describes the **Cat Food Reviews** project architecture, code style preferences, and context to use in future sessions. Code style was inferred from the codebase; edit this file to reflect your own preferences.

---

## Project Overview

**Cat Food Reviews** is a static marketing site for showcasing top-rated cat food products. It includes:

- A **Next.js 14** app (React, TypeScript, Tailwind) served via `npm run dev` at `http://localhost:3000`
- A **standalone `index.html`** that can be opened directly in a browser (no server)

Sections: **Hero** → **Top Foods** (4 products) → **About** → **FAQ** (accordion) → **Contact** (Instagram, Facebook Marketplace).

---

## Project Architecture

### Directory Structure

```
cat-food-reviews/
├── public/
│   └── images/              # Static assets (hero-cat.jpg, food-1..4.jpg)
├── src/
│   ├── app/                 # Next.js App Router
│   │   ├── layout.tsx       # Root layout, metadata, fonts
│   │   ├── page.tsx         # Home page — composes all sections
│   │   └── globals.css      # Tailwind imports, base styles
│   ├── components/          # React components
│   │   ├── Hero.tsx         # Hero section (background image, tagline)
│   │   ├── TopFoods.tsx     # Grid of 4 food cards
│   │   ├── FoodCard.tsx     # Single food card (image, name, brand, stars, description)
│   │   ├── About.tsx        # Mission / about copy
│   │   ├── FAQ.tsx          # FAQ container; maps over faqs data
│   │   ├── FAQItem.tsx      # Single accordion item ('use client')
│   │   └── Contact.tsx      # Social links (Instagram, Facebook Marketplace)
│   ├── data/
│   │   ├── foods.ts         # Array of Food objects
│   │   └── faqs.ts          # Array of FAQ objects
│   └── types/
│       └── index.ts         # Food, FAQ interfaces
├── index.html               # Standalone HTML version (Tailwind CDN, no Next.js)
├── tailwind.config.js       # Theme (primary orange palette)
├── tsconfig.json            # Path alias @/* → ./src/*
└── package.json
```

### Key Conventions

- **Path alias**: `@/` → `src/` (e.g. `import { Food } from '@/types'`, `import Hero from '@/components/Hero'`).
- **Data**: Content lives in `src/data/` (foods, FAQs). Components import from there; no API calls.
- **Types**: Shared interfaces in `src/types/index.ts`. Data modules use these types.
- **Images**: Next.js app expects images in `public/images/` (e.g. `hero-cat.jpg`, `food-1.jpg`). Hero can use an external URL or local file.

### Tech Stack

- **Next.js 14** (App Router), **React 18**, **TypeScript** (strict)
- **Tailwind CSS** for styling
- **react-icons** (e.g. `FaInstagram`, `FaFacebook`)

---

## Code Style Preferences

### TypeScript & React

- Use **TypeScript** for all new code. Keep `strict: true`.
- **Functional components** only. Use **default export** for components.
- Define **props interfaces** as `ComponentNameProps` (e.g. `FoodCardProps`, `FAQItemProps`) in the same file as the component.
- Use **`Readonly<{ ... }>`** for layout/root-level props when appropriate (see `layout.tsx`).

### Components & Files

- **One main component per file.** Small helpers (e.g. `renderStars`) can live inside the component.
- Use **`'use client'`** only when needed (e.g. `useState` in `FAQItem`). Prefer server components by default.
- **Component names**: PascalCase. **File names**: PascalCase for components (`Hero.tsx`, `FoodCard.tsx`).

### Styling (Tailwind)

- Use **Tailwind utility classes** only. No custom CSS unless necessary (e.g. globals).
- **Responsive**: Prefer `md:` (and `lg:` when useful) for breakpoints. Mobile-first.
- **Spacing**: Use Tailwind spacing scale (e.g. `p-4`, `mb-6`, `gap-6`). Sections use `py-16 md:py-20`.
- **Colors**: Prefer `gray-*` for neutrals, `primary-*` for brand (see `tailwind.config.js`). Use semantic color names only when they map to theme (e.g. `text-gray-800`).
- **Transitions**: Use `transition-*` for hover/focus (e.g. `transition-colors`, `transition-shadow duration-300`).

### Data & Types

- **Data** in `src/data/*.ts`: export arrays of typed objects (e.g. `foods: Food[]`, `faqs: FAQ[]`).
- **Types** in `src/types/index.ts`: export interfaces (e.g. `Food`, `FAQ`). Use consistent property names (`id`, `name`, etc.).

### Accessibility & Semantics

- Use **semantic HTML** (`<section>`, `<main>`, `<h1>`–`<h3>`, etc.).
- **Images**: always `alt` text. Decorative or background images: `role="img"` and `aria-label` when applicable.
- **Links**: `target="_blank"` → `rel="noopener noreferrer"`. Add `aria-label` where helpful (e.g. “Visit our Instagram”).
- **Interactive UI** (e.g. FAQ accordion): `aria-expanded`, focus styles (`focus:ring-2 focus:ring-primary-500`), keyboard-friendly.

### General

- **Comments**: Use sparingly. Prefer clear naming. Add short comments for non-obvious behavior (e.g. image fallback).
- **Formatting**: Prefer consistent indentation and line breaks. Use double quotes for JSX strings where applicable.

---

## Context for Future Sessions

### Running the Project

- **Next.js app**: `npm install` → `npm run dev` → open `http://localhost:3000`.
- **Standalone**: Open `index.html` in a browser (no install required).

### Where to Change Things

| Change | Location |
|--------|----------|
| Top 4 foods | `src/data/foods.ts` |
| FAQ items | `src/data/faqs.ts` |
| Hero tagline / background | `src/components/Hero.tsx` |
| About / mission | `src/components/About.tsx` |
| Social links | `src/components/Contact.tsx` |
| Site metadata (title, description) | `src/app/layout.tsx` |
| Theme colors | `tailwind.config.js` |
| Global styles | `src/app/globals.css` |

### Adding Features

- **New sections**: Add a component under `src/components/`, then import and render it in `src/app/page.tsx`.
- **New data types**: Add interfaces in `src/types/index.ts`, then data in `src/data/` and components as needed.
- **New pages**: Use App Router under `src/app/` (e.g. `src/app/about/page.tsx`).

### Standalone `index.html`

- Mirrors the Next.js layout and content. Uses Tailwind and Font Awesome via CDN.
- Hero uses an **embedded SVG data URI** so it works when opening the file directly (`file://`). External image URLs often fail in that context.

---

## Summary

- **Architecture**: Next.js 14 App Router, `src/app` + `src/components` + `src/data` + `src/types`, path alias `@/*`.
- **Style**: TypeScript strict, functional components, Tailwind-only styling, semantic HTML, accessibility considered.
- **Context**: Data-driven presentational site; edit data and components for content, `layout` for metadata, Tailwind config for theme.

Use this file in future sessions to align suggestions with the project’s structure and preferences.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/breakthr-oss)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/breakthr-oss)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
