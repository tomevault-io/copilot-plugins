## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio website built with React 17 using Create React App. The application showcases personal information, education, skills, training, and interests through a single-page application with animated intro and navigation.

## Development Commands

- `npm start` - Start development server on http://localhost:3000
- `npm run build` - Build for production (outputs to `build/` folder)
- `npm test` - Run tests in interactive watch mode
- `npm run eject` - Eject from Create React App (irreversible)

## Architecture & Structure

### Application Flow
- **Intro Component**: Displays animated name letters "JAD ALMAALOUF" with a 13-second delay before showing navigation
- **Routing**: Uses React Router v5 with conditional rendering - navigation only appears after the intro animation
- **Navigation State**: Uses multiple useState hooks to track active navigation states (activeP, activeE, activeT, activeS, activeI)

### Key Components Structure
```
src/
├── App.js (Main app with routing and 13-second menu delay)
├── index.js (BrowserRouter wrapper)
└── components/
    ├── intro.js (Animated name display + navigation menu)
    ├── personnal.js (Personal information with profile image)
    ├── education.js (Education details)
    ├── skills.js (Skills showcase)
    ├── training.js (Training/certifications)
    ├── interests.js (Personal interests)
    └── footer.js (Footer component)
```

### Routing Pattern
- Uses React Router v5 (`react-router-dom@^5.2.0`)
- Routes are conditionally rendered after a 13-second timer
- Navigation state management uses individual useState hooks for each section
- Active/inactive classes are applied based on current route

### Styling & Assets
- CSS styling in `App.css`
- Images stored in `src/images/` (profile pictures, social media icons, interest icons)
- Uses conditional CSS classes for active navigation states

## Notable Implementation Details

- **Delayed Navigation**: Both App.js and intro.js implement identical 13-second delays before showing menu/routes
- **State Management**: Navigation uses individual boolean states rather than a single state object
- **Personal Information**: Contains actual contact details and personal information
- **Image Handling**: Profile image imported and used as React component prop

## Dependencies

Key dependencies include:
- React 17.0.2 with React DOM
- React Router DOM 5.2.0 for routing
- Create React App 5.0.1 for build tooling
- Testing Library suite for testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jad-maalouf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
