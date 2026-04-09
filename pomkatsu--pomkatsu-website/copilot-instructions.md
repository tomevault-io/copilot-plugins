## pomkatsu-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Landing page website for Pomkatsu LLC, a Texas-based company that develops mobile applications for document translation and language learning. Built with Vue 3, Vite, and Tailwind CSS, deployed on Firebase Hosting.

## Design System

### Color Palette
Our brand uses a warm, professional color scheme with brown and cream tones:

#### Primary Colors (Brown tones)
- **primary**: `#46301C` - Main brown color for headings and primary text
- **primary-light**: `#5A3E26` - Lighter brown for body text and secondary elements
- **primary-dark**: `#2E1F12` - Dark brown for emphasis and strong text

#### Secondary Colors (Cream tones)
- **secondary**: `#FCF1DC` - Main cream background color
- **secondary-light**: `#FFF8ED` - Light cream for highlighted sections
- **secondary-dark**: `#F5E5C8` - Darker cream for borders and subtle backgrounds

#### Accent Colors (Pomkatsu pink - use sparingly)
- **pomkatsu-200**: `#fecdd3` - Light pink for critical warnings background
- **pomkatsu-700**: `#be123c` - Medium pink for warning borders
- **pomkatsu-900**: `#881337` - Dark pink for warning text

#### Legal/Documentation Colors
- **legal-text**: `#5A3E26` - Standard text in legal documents
- **legal-heading**: `#46301C` - Headings in legal documents
- **legal-border**: `#F5E5C8` - Borders and dividers
- **legal-warning**: `#be123c` - Warning indicators
- **legal-warning-light**: `#fecdd3` - Warning backgrounds
- **legal-warning-dark**: `#881337` - Warning text emphasis

### Design Principles

1. **Flat Design**: No drop shadows on primary UI elements (navigation, headers)
2. **Transparency**: Use semi-transparent backgrounds where appropriate (e.g., table of contents)
3. **Consistency**: All text uses the brown color palette, pink is reserved for warnings only
4. **Readability**: High contrast between text and backgrounds, especially in legal documents
5. **Minimalism**: Clean, uncluttered layouts with ample whitespace

### Typography
- Font family: Inter (primary), system fonts as fallback
- Headings: Bold weight, primary brown color
- Body text: Regular weight, primary-light brown color
- Legal text: Enhanced line height (1.75) for readability

### Component Patterns

#### Legal Documents
- Transparent header boxes with subtle borders
- Color-coded information boxes:
  - Info/highlight boxes: Light cream background
  - Warning boxes: Light cream with pink border
  - Critical warnings: Light pink background with dark text
  - Disclaimers: Cream with dashed border
- Sticky table of contents on desktop (semi-transparent cream)
- Responsive layout that works on all devices

#### Navigation
- Fixed position, flat design (no shadows)
- Cream background color
- Brown text with hover states

### File Structure
- `/src/views/` - Vue page components
- `/src/components/` - Reusable Vue components
- `/src/legal/` - Markdown files for legal content
- `/public/` - Static assets and favicons

## Essential Commands

```bash
npm install          # Install dependencies
npm run dev          # Start development server at http://localhost:5173
npm run build        # Build for production (outputs to dist/)
npm run preview      # Preview production build locally
npm run deploy       # Build and deploy to Firebase Hosting (requires firebase login)
```

## Architecture

### Project Structure
```
src/
├── views/           # Page components (HomePage, TermsOfService, etc.)
├── components/      # Reusable components (LegalLayout, ContactForm)
├── legal/          # Markdown files for legal content
├── router/         # Vue Router configuration
├── style.css       # Global styles and Tailwind imports
└── main.js         # App entry point
```

### Routing Architecture
- Single Page Application using Vue Router with HTML5 history mode
- All routes defined in `src/router/index.js`
- Firebase configured with SPA rewrites (all routes → index.html)
- Legal pages share common layout via `LegalLayout` component wrapper

### Component Architecture

#### LegalLayout Component
Wrapper component for all legal pages providing:
- Fixed navigation header (flat design, no shadows)
- Footer with links to all legal pages
- Contact form modal integration
- Responsive breakpoints (mobile/desktop)
- Print-friendly styles

#### Legal Page Pattern
Each legal page follows this structure:
```vue
<script setup>
import LegalLayout from '../components/LegalLayout.vue'
</script>

<template>
  <LegalLayout title="Page Title">
    <!-- Content here -->
  </LegalLayout>
</template>
```

### Styling Architecture
- Tailwind CSS for utility classes
- Custom color variables in `tailwind.config.js`
- Component-scoped styles for complex layouts
- Design principles: Flat UI, no shadows on primary elements, brown/cream color scheme

### Build & Deployment
- Vite handles bundling with automatic code splitting
- Firebase Hosting serves from `dist/` directory
- Cache headers configured: JS/CSS (1 year), images (1 day)
- SPA routing handled via Firebase rewrites

## Legal Content Management

Legal documents exist in two forms:
1. **Markdown source**: `/src/legal/[document-name].md` - Raw content
2. **Vue component**: `/src/views/[DocumentName].vue` - Styled presentation

When updating legal content:
- Edit markdown for content changes
- Edit Vue component for styling/layout changes
- Maintain consistency across all legal pages using established patterns

## Contact Information
- Support email: support@pomkatsu.com
- Company: Pomkatsu LLC (Texas)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pomkatsu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pomkatsu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
