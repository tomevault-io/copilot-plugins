## ecommerce-lite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DreamBed is an e-commerce landing page for pillows and bedding products (almohadas y ropa de cama) built with Astro. It features a quote-based system where customers add products to a cart and send the quote request via WhatsApp instead of direct checkout.

## Development Commands

```bash
# Install dependencies
npm install

# Start development server (usually runs on http://localhost:4321)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## Technology Stack

- **Framework**: Astro 5.16.6 (Static Site Generation)
- **Styling**: Tailwind CSS 4.1.18 with custom theme configuration
- **Language**: TypeScript (used in Astro components via frontmatter)
- **Font**: Inter (loaded from Google Fonts)

## Architecture & Key Concepts

### Component-Based Structure

The application follows Astro's component architecture with `.astro` files that combine HTML-like templates with TypeScript frontmatter:

```
src/
├── components/     # Reusable UI components
├── data/          # JSON data files for products
├── layouts/       # Page layout templates
├── pages/         # Route pages (index.astro = homepage)
└── styles/        # Global styles
```

### Cart System Architecture

The cart is a **client-side quote system** that:

1. **Storage**: Uses localStorage to persist cart data between sessions
2. **UI Components**:
   - `Header.astro`: Contains cart button with counter badge (line 44-49)
   - `CartModal.astro`: Modal overlay for cart display
   - `ProductCard.astro`: "Agregar" button with data attributes (lines 24-31)
3. **Client Scripts**: Cart logic is implemented in `<script>` tags within components (see Header.astro:87-143)
4. **WhatsApp Integration**: Generates formatted quote message with product list and total, sent via WhatsApp Web/App

### Product Data Flow

Products are centrally managed in `src/data/Products.json` and imported in `Products.astro`:

```typescript
import productsData from '../data/Products.json';
```

Each product has: `id`, `name`, `description`, `price`, `image` (Unsplash URLs), `category`

### Styling System

- **Tailwind Config** (`tailwind.config.mjs`): Defines custom colors (primary: #667eea, secondary: #764ba2), font family (Inter), and gradient backgrounds
- **Global Styles** (`src/styles/globals.css`): Uses `@layer base` for reset and typography defaults
- **Utility-First**: Components use Tailwind utility classes extensively

### Layout Pattern

All pages use `Layout.astro` which provides:
- SEO meta tags (title, description)
- Google Fonts preconnect optimization
- Header, Footer, and main content slot
- Spanish language (`lang="es"`)

## Important Configuration Details

### WhatsApp Number

The WhatsApp phone number needs to be configured. According to the README, it should be in `src/pages/index.astro` line 391, though current implementation may vary. Format: country code + number without spaces (e.g., `5215512345678` for Mexico +52 1 55 1234 5678).

### Tailwind Configuration

The project uses **Tailwind CSS v4**, which has different configuration:
- PostCSS-based (`postcss.config.mjs`)
- Imports via `@import "tailwindcss"` in globals.css
- Content scanning: `['./src/**/*.{astro,html,js,jsx,md,mdx,svelte,ts,tsx,vue}']`

### Theme Customization

Brand colors are defined in `tailwind.config.mjs`:
- Primary gradient: `#667eea` to `#764ba2`
- Dark text: `#2c3e50`
- Light background: `#f8f9fa`

To change branding, update these values and the company name in `Header.astro` (line 9) and `Hero.astro`.

## Development Patterns

### Astro Component Structure

```astro
---
// TypeScript frontmatter (server-side, build time)
import Component from './Component.astro';
interface Props { /* ... */ }
const { prop } = Astro.props;
---

<!-- HTML template -->
<div>{prop}</div>

<script>
  // Client-side JavaScript (runs in browser)
  // Executes once per page load
</script>
```

### Adding New Products

Edit `src/data/Products.json` with the structure:
```json
{
  "id": 10,
  "name": "Product Name",
  "description": "Detailed description",
  "price": 99.99,
  "image": "https://... or /images/filename.jpg",
  "category": "Category"
}
```

For local images, place them in `public/images/` and reference as `/images/filename.jpg`.

## Common Tasks

### Modifying the Product Grid

Products are rendered in `Products.astro` using `.map()` over the products array (line 14-16). Each product is passed to `ProductCard.astro` as props.

### Updating Contact Information

Footer contact details are in `src/components/Footer.astro` (lines 18-23 per README).

### Changing Hero Section

Main headline and CTA are in `src/components/Hero.astro` (line 6 per README).

### Responsive Design

The project uses Tailwind's responsive prefixes (`md:`, `lg:`) extensively. Key breakpoints:
- Mobile-first approach
- `md:` prefix for tablet/desktop
- Mobile menu toggle in Header (lines 61-66, 71-84)

## Notes for Development

- **No Backend**: This is a static site. Product data is in JSON, no database.
- **No Tests**: No testing framework is configured in package.json.
- **No Linting**: No ESLint or Prettier configuration present.
- **Image Hosting**: Currently uses Unsplash URLs; consider migrating to local assets for production.
- **WhatsApp-Only**: No payment gateway or checkout flow; designed for quote requests.
- **Spanish Language**: All content and UI text is in Spanish.

## Build Output

Astro builds to `dist/` directory by default. The build is optimized with:
- Static HTML generation
- Automatic image optimization
- CSS bundling and minification
- Client-side hydration only where needed (interactive components)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sebahernandez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
