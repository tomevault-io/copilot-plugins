## pofo

> A personal portfolio website built with Next.js.

# Portfolio Site

A personal portfolio website built with Next.js.

## Tech Stack

- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Deployment**: Vercel (recommended)

## Project Structure

```
src/
├── app/              # App Router pages and layouts
│   ├── layout.tsx    # Root layout
│   ├── page.tsx      # Home page
│   └── */            # Other routes
├── components/       # Reusable UI components
├── lib/              # Utility functions and helpers
└── styles/           # Global styles
public/               # Static assets (images, fonts, etc.)
```

## Commands

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## Development Guidelines

### Code Style
- Use functional components with TypeScript
- Prefer named exports for components
- Use Tailwind CSS for styling (avoid inline styles and CSS modules)
- Follow Next.js App Router conventions

### Component Structure
- Keep components small and focused
- Place shared components in `src/components/`
- Colocate page-specific components within their route folders

### Performance
- Use Next.js Image component for images
- Implement lazy loading for below-fold content
- Prefer server components where possible

### Git Commits
- Use conventional commit messages (feat:, fix:, docs:, etc.)

## Design Guide

### Color Palette
- **Background**: Dark theme (`#0a0a0a` or `bg-neutral-950`)
- **Text Primary**: White (`#ffffff`)
- **Text Secondary**: Gray (`#a3a3a3` or `text-neutral-400`)
- **Accent Primary**: Yellow/Gold (`#facc15` or `bg-yellow-400`)
- **Accent Secondary**: White outline for secondary elements
- **Glow Effects**: Use subtle white/gray gradients for depth

### Typography
- **Headlines**: Bold, large (text-4xl to text-6xl), tight line-height
- **Body**: Regular weight, readable size (text-base to text-lg)
- **Font Family**: Modern sans-serif (Inter, system-ui)
- **Hierarchy**: Clear distinction between headings and body text

### Layout
- **Spacing**: Generous whitespace, section padding (py-16 to py-24)
- **Container**: Centered with max-width, horizontal padding (px-4 to px-8)
- **Grid**: Two-column hero layout (content left, visual right)
- **Navigation**: Fixed top - logo left, nav center, auth buttons right

### Components

#### Buttons
- **Primary**: Filled yellow/gold background, black text, rounded-full, hover effect
- **Secondary**: Transparent with white border, white text, rounded-full
- **Icon buttons**: Include arrow icon (→) for CTAs

#### Navigation
- Minimal, clean design
- Transparent/dark background
- Dropdown menus with chevron indicators
- Login/Signup buttons on the right

#### Hero Section
- Large bold headline (2-3 lines max)
- Supporting subtitle text in gray
- CTA buttons group (primary + secondary)
- Trust badges/social proof below CTAs
- Abstract visual element on the right side

#### Visual Elements
- Abstract geometric shapes (hexagons, nested polygons)
- Glow/blur effects for depth (`blur-xl`, `opacity-50`)
- Gradient overlays for visual interest
- Dark cards with subtle borders

#### Social Proof
- Company logos in grayscale
- Horizontal layout with even spacing
- Subtle opacity to not distract from main content

### Animation & Interactions
- Subtle hover transitions (duration-200 to duration-300)
- Smooth scroll behavior
- Fade-in animations for page load
- Scale/opacity changes on interactive elements

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/clwmfksek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
