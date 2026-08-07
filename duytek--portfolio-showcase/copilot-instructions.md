## portfolio-showcase

> - Moodboard: (`.github/.instructions/.moodboard`)

# GitHub Copilot Instructions

## References

- Moodboard: (`.github/.instructions/.moodboard`)

## Project Overview

This is a personal portfolio website built with React 18 + TypeScript + Vite, showcasing a Software Developer's experience. It features a dark theme with custom Material-UI styling, VS Code-inspired design, and comprehensive SEO optimization.

## Architecture & Key Patterns

### Provider Pattern Architecture

- `App.tsx` → `AppProviders` → `RouterProvider` chain
- Material-UI `ThemeProvider` and `CssBaseline` wrapped in a composition pattern
- Theme configuration centralized in `src/config/theme/theme.ts` with custom dark palette

### Routing & Layout Structure

- React Router with nested routes in `src/config/routes/routes.ts`
- Single `PageLayout` component with sticky tab navigation
- Content rendered via `<Outlet />` with responsive padding system
- Navigation tabs simulate VS Code-style interface

### SEO Implementation

- **Comprehensive SEO component** (`src/components/SEO.tsx`) that dynamically updates meta tags, structured data, and canonical URLs per route
- **Route-specific JSON-LD** structured data for Person, ItemList, and Occupation schemas
- **Static SEO** in `index.html` with Open Graph, Twitter Cards, and default structured data
- **SEO assets**: sitemap.xml, robots.txt, web manifest in `/public`

### Material-UI Customization Patterns

- **Custom theme** with VS Code-inspired dark colors (`#1E1E1E`, `#252526`)
- **Roboto Mono** font family for developer aesthetic
- **styled() components** with responsive breakpoints using theme.breakpoints
- **Component overrides** for buttons (no text transform, custom colors)

## Development Workflow

### Key Commands

```bash
npm run dev          # Vite dev server with HMR
npm run build        # TypeScript compilation + Vite build
npm run lint         # ESLint with TypeScript rules
npm run preview      # Preview production build
```

### File Organization Conventions

```
src/
├── components/      # Reusable components (SEO)
├── config/         # Configuration modules
│   ├── constants/  # App constants (fullName, title, descriptions)
│   ├── routes/     # React Router configuration
│   └── theme/      # Material-UI theme customization
├── pages/          # Route components organized by feature
│   ├── layout/     # PageLayout, Tabs components
│   ├── home/       # Home, Contacts, Logos components
│   ├── projects/   # Projects page
│   ├── experience/ # Experience page
│   └── skill/      # Skills page
├── providers/      # Context providers and app setup
├── styles/         # Global CSS (mobile optimizations)
└── assets/         # Static assets (SVG icons)
```

## Critical Implementation Details

### Responsive Design System

- **Mobile-first** breakpoints using Material-UI system
- **Touch targets** minimum 44px on mobile (mobile.css)
- **Custom scrollbar** styling with `.custom-scrollbar` class
- **Responsive padding** pattern: `theme.spacing(3, 7)` → `theme.spacing(2, 3)` → `theme.spacing(1.5, 2)`

### SEO Component Usage

Every page should include:

```tsx
import SEO from '../components/SEO';

function PageComponent() {
	return (
		<>
			<SEO
				title="Page Title"
				description="Page description"
				keywords="additional, keywords"
			/>
			{/* Page content */}
		</>
	);
}
```

### Theme Integration Patterns

Use Material-UI's styled API with theme access:

```tsx
const StyledComponent = styled('div')(({ theme }) => ({
	padding: theme.spacing(2),
	[theme.breakpoints.down('sm')]: {
		padding: theme.spacing(1),
	},
}));
```

### Constants Usage

Import app constants from `src/config/constants/index.ts`:

```tsx
import { constants } from '../config/constants';
// Access: constants.fullName, constants.title, constants.descriptions
```

## Design System

### Color Palette

- **Primary**: `#37B7C3` (teal), **Dark**: `#088395`, **Light**: `#EBF4F6`
- **Background**: `#1E1E1E` (main), `#252526` (paper/cards)
- **Text**: `#ffffff` (primary), `#d0d0d0` (secondary)

### Typography Scale

- **Font**: Roboto Mono for developer aesthetic
- **H1**: `4.25rem` responsive with clamp()
- **Base**: `14px` with responsive scaling

## Dependencies & Integration

### Core Stack

- **React 18** with TypeScript
- **Material-UI v7** with Emotion styling
- **React Router DOM v6** for routing
- **React Spring** for animations
- **Vite** for build tooling

### SEO & Performance

- **Structured data** automatically updates per route
- **Preconnect** to fonts.googleapis.com in index.html
- **Web manifest** for PWA capabilities
- **Performance optimizations** in mobile.css

## Special Considerations

### Mobile Optimizations

- Input font-size: 16px to prevent iOS zoom
- Tap highlight disabled via `-webkit-tap-highlight-color: transparent`
- Overflow-x hidden to prevent horizontal scroll
- Touch scrolling enabled with `-webkit-overflow-scrolling: touch`

### VS Code Theme Integration

The project includes Liquid Glass theme documentation in `/moodboard/` with ethereal glass effects and backdrop blur patterns - reference these files when implementing glass-like UI components.

### Build Configuration

- ESM-only project (`"type": "module"` in package.json)
- TypeScript strict mode with separate configs for app and node
- SWC for fast refresh instead of Babel

---
> Source: [DuyTek/portfolio-showcase](https://github.com/DuyTek/portfolio-showcase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
