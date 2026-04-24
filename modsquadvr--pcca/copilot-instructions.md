## pcca

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React-based website for PCCA (Pacific Climate Conversation Alliance) built with Create React App. The project showcases various research projects, provides information about the organization, and includes interactive features like mapping and hackathon information.

## Build and Development Commands

- `npm start` - Start development server (runs on http://localhost:3000)
- `npm run build` - Build for production
- `npm test` - Run tests in interactive watch mode
- `npm run eject` - Eject from Create React App (one-way operation)

Note: This project uses `react-app-rewired` instead of standard `react-scripts` to allow custom Babel configuration.

## Architecture

### Core Structure
- **Entry Point**: `src/index.js` - React root with Material-UI ThemeProvider and CssBaseline
- **Routing**: `src/Router.js` - React Router configuration with routes for Home, About, Projects, Podcasts, Mapping, and Hackathon pages
- **Theme**: `src/assets/theme.js` - Material-UI theme configuration
- **Components**: `src/components/` - Reusable components like FadeRule
- **Pages**: `src/pages/` - Main page components organized by feature

### Key Technologies
- **React 18** with React Router DOM for navigation
- **Material-UI (MUI)** for UI components and theming
- **FontAwesome** for icons
- **React Responsive Carousel** for image carousels
- **Babel** with custom configuration via `.babelrc` and `config-overrides.js`

### Page Structure
- **Home** (`src/pages/Home/`) - Landing page with sections for Projects and Services
- **About** (`src/pages/About/`) - Organization information
- **Projects** (`src/pages/Projects/`) - Project showcase with individual project components in `ProjectList/`
- **Podcasts** (`src/pages/Podcasts/`) - Podcast content
- **Mapping** (`src/pages/mapping/`) - Interactive mapping features with PDF popup functionality
- **Hackathon** (`src/pages/hackathon/`) - Hackathon information

### Asset Organization
- **Logos**: `src/assets/logos/` - Various PCCA logos in different sizes and colors
- **Photos**: `src/assets/photos/` - Project images, headshots, and general imagery
- **Project Assets**: `src/assets/photos/projects/` - Organized by project name

### Custom Configuration
- Uses `react-app-rewired` with `customize-cra` for Babel customization
- Babel configuration in `.babelrc` includes module resolver plugin
- Custom babel plugin for private property syntax support

## Project-Specific Features

### Dynamic Project Routing
Projects can be accessed via `/projects` (shows all) or `/projects/:slug` (shows specific project).

### Modal System
Projects use a CardModal component for detailed project information display.

### PDF Integration
The mapping page includes PDF popup functionality for document viewing.

### Responsive Design
Uses Material-UI's responsive breakpoint system and CSS for mobile-friendly layouts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/modsquadvr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
