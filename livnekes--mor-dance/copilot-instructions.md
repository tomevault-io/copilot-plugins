## mor-dance

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React + TypeScript + Vite project for "Elegance Dance Studio" - a dance studio website. The application uses Material-UI (MUI) for the component library and styling, with a custom purple/pink color theme.

## Development Commands

- `npm run dev` - Start development server with hot reload
- `npm run build` - Build for production (runs TypeScript compiler then Vite build)
- `npm run lint` - Run ESLint on the codebase
- `npm run preview` - Preview production build locally

## Architecture & Structure

### Key Technologies
- **React 19** with TypeScript
- **Vite** for bundling and dev server
- **Material-UI (MUI)** for components and theming
- **Emotion** for styling (comes with MUI)

### Theme Configuration
The app uses a custom MUI theme defined in `src/main.tsx:8-40` with:
- Primary color: Purple (#8e24aa)
- Secondary color: Soft pink (#f8bbd9)
- Custom typography using Inter font family
- Light background theme

### Application Structure
- **Single-page application**: All content is in `src/App.tsx` as one large component
- **No routing**: Currently a static landing page
- **Component structure**: Uses MUI's Container, Grid, Card, AppBar, and other components
- **Styling approach**: Primarily using MUI's sx prop for inline styles

### Key Components & Sections
- Navigation bar with responsive design
- Hero section with gradient background
- Features section (Why Choose Elegance)
- Classes section with dance class cards
- About section
- Footer with contact information

## Development Notes

### ESLint Configuration
- Uses modern ESLint config with TypeScript support
- Includes React Hooks and React Refresh plugins
- Ignores `dist` folder

### TypeScript Configuration
- Uses project references with separate configs for app and node
- App config: `tsconfig.app.json`
- Node config: `tsconfig.node.json`

### Testing
No testing framework is currently configured in this project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/livnekes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/livnekes)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
