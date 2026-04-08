## surajfale

> This document provides comprehensive context and guidelines for AI agents working on this repository.

# AI Agent Guide for Suraj Fale Portfolio

This document provides comprehensive context and guidelines for AI agents working on this repository.

## 1. Project Identity
- **Name**: Suraj Fale Portfolio
- **Purpose**: Professional portfolio website for Suraj Fale, a Principal Engineer specializing in Scala, Kafka, and Distributed Systems.
- **Type**: Single Page Application (SPA)
- **Live URL**: https://surajfale.netlify.app

## 2. Technology Stack
- **Core**: React 18, TypeScript 5.3
- **Build Tool**: Vite 5.1
- **Package Manager**: pnpm 8+ (Strictly enforce this)
- **UI Framework**: Material-UI (MUI) v5
- **Styling Engine**: Emotion (CSS-in-JS)
- **Routing**: React Router DOM v7 (though mostly single-page scroll)
- **Linting**: ESLint 9 (Flat Config)
- **Deployment**: Netlify

## 3. Project Structure
```
src/
├── components/        # UI Components
│   ├── Hero.tsx      # Landing section
│   ├── About.tsx     # Bio & Skills
│   ├── Projects.tsx  # Portfolio showcase
│   ├── Socials.tsx   # Contact/Social links
│   ├── Footer.tsx    # Site footer
│   └── ThemeToggle.tsx # Dark/Light mode switch
├── content/
│   └── profile.ts    # CENTRAL SOURCE OF TRUTH for all text/data
├── pages/            # Route components (if any)
├── utils/            # Helper functions
├── App.tsx           # Main layout & ThemeProvider setup
├── theme.ts          # MUI Theme configuration (Colors, Typography)
└── main.tsx          # App entry point
```

## 4. Development Workflow
- **Install Dependencies**: `pnpm install`
- **Start Dev Server**: `pnpm dev` (Runs on http://localhost:5173)
- **Build for Production**: `pnpm build`
- **Preview Build**: `pnpm preview`
- **Lint Code**: `pnpm lint`

## 5. Design System & Aesthetics
- **Theme**: Supports Light and Dark modes (persisted in localStorage).
- **Primary Colors**:
  - LinkedIn Blue: `#0077B5`
  - GitHub Dark: `#24292E`
- **Visual Style**:
  - **Glassmorphism**: Heavy use of semi-transparent backgrounds with blur.
  - **Gradients**: Modern, vibrant gradients for text and backgrounds.
  - **Animations**: Smooth transitions, hover effects (scale, lift), and micro-interactions.
- **Typography**: Modern sans-serif (via MUI default or configured font).

## 6. Content Management
**CRITICAL**: Do not hardcode text in components.
- All personal data, project details, and social links are stored in `src/content/profile.ts`.
- To update the portfolio content, modify `src/content/profile.ts`.
- Interfaces for content are defined in the same file to ensure type safety.

## 7. Agent Guidelines & Rules
1.  **Package Manager**: ALWAYS use `pnpm`. Do not use `npm` or `yarn`.
2.  **Styling**: Use MUI's `sx` prop or `styled` components. Avoid plain CSS files unless for global resets (`index.css`).
3.  **Icons**: Use `@mui/icons-material`.
4.  **Responsiveness**: Always ensure designs work on Mobile, Tablet, and Desktop. Use MUI's `useMediaQuery` or responsive `sx` values (e.g., `width: { xs: '100%', md: '50%' }`).
5.  **Code Quality**: Run `pnpm lint` before finishing a task to ensure no linting errors.
6.  **File Naming**: PascalCase for components (`MyComponent.tsx`), camelCase for utilities (`myUtility.ts`).
7.  **Privacy**: NEVER add personal contact info (phone, email, address) directly to the code.

## 8. Deployment
- The project is deployed on Netlify.
- Configuration is in `netlify.toml`.
- Deploys automatically on push to `main`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/surajfale)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/surajfale)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
