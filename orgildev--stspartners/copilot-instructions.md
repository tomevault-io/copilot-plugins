## stspartners

> This project is a modern, responsive website for **S&TS Partners LLC**, a professional law firm based in Mongolia. The website highlights the firm's practice areas, partner qualifications, and provides a contact interface for potential clients.

# GEMINI.md - S&TS Partners LLC Website

## Project Overview
This project is a modern, responsive website for **S&TS Partners LLC**, a professional law firm based in Mongolia. The website highlights the firm's practice areas, partner qualifications, and provides a contact interface for potential clients.

### Core Technologies
- **Framework:** [React 19](https://react.dev/)
- **Build Tool:** [Vite 8](https://vitejs.dev/)
- **Language:** [TypeScript](https://www.typescriptlang.org/)
- **Animation:** [Framer Motion](https://www.framer.com/motion/)
- **Icons:** [Lucide React](https://lucide.dev/)
- **Styling:** Vanilla CSS with CSS Variables for theme management.

### Architecture
- **Entry Point:** `src/main.tsx`
- **Main Component:** `src/App.tsx` (Single-page architecture with smooth-scroll sections)
- **Styles:** `src/index.css` contains the global theme, typography, and layout rules.
- **Assets:** Managed in `src/assets/` and `public/`.

## Building and Running

### Development
To start the development server with Hot Module Replacement (HMR):
```bash
npm run dev
# or
npm start
```

### Production
To build the project for production (outputs to `dist/`):
```bash
npm run build
```
To preview the production build locally:
```bash
npm run preview
```

### Linting
To run ESLint:
```bash
npm run lint
```

## Development Conventions

### Styling
- **Theme Variables:** Always use CSS variables defined in `:root` within `src/index.css` (e.g., `var(--primary)`, `var(--secondary)`) to maintain brand consistency.
- **Responsive Design:** Use the `.container` class for centered layouts and media queries for mobile adjustments (breakpoint is typically 768px).
- **Animations:** Use `framer-motion` for scroll-reveal and transition effects to ensure a premium feel.

### Components
- **Functional Components:** Use React Functional Components (`React.FC`) with TypeScript interfaces for props where applicable.
- **Icons:** Use `lucide-react` for all UI icons.

### Assets
- **Images:** High-quality professional images should be placed in `src/assets/`.
- **Icons/Favicons:** SVG and PNG icons for browser metadata are located in `public/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/orgildev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/orgildev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
