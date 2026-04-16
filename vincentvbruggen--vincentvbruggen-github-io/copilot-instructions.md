## vincentvbruggen-github-io

> This is a personal portfolio website for a game developer named Vincent. It showcases his projects, skills, and provides contact information. The project is a single-page application built with React, TypeScript, and Vite. It uses `react-router-dom` for routing and `styled-components` for styling, with a custom theme. The portfolio is designed to be visually appealing, with animations and particle effects from `framer-motion` and `tsparticles`.

# GEMINI.md

## Project Overview

This is a personal portfolio website for a game developer named Vincent. It showcases his projects, skills, and provides contact information. The project is a single-page application built with React, TypeScript, and Vite. It uses `react-router-dom` for routing and `styled-components` for styling, with a custom theme. The portfolio is designed to be visually appealing, with animations and particle effects from `framer-motion` and `tsparticles`.

## Building and Running

### Prerequisites

*   Node.js and npm should be installed.

### Installation

To install the dependencies, run:

```bash
npm install
```

### Development

To start the development server, run:

```bash
npm run dev
```

This will start the application on `http://localhost:5173` by default.

### Building

To build the application for production, run:

```bash
npm run build
```

The built files will be in the `dist` directory.

### Preview

To preview the production build locally, run:

```bash
npm run preview
```

### Linting

To lint the code, run:

```bash
npm run lint
```

## Development Conventions

### Project Structure

The project follows a standard React project structure:

*   `src/`: Contains all the source code.
*   `src/components/`: Contains reusable React components.
*   `src/data/`: Contains the project data.
*   `src/projects/`: Contains the individual project pages.
*   `src/sections/`: Contains the different sections of the main page (About, Contact, etc.).
*   `src/styles/`: Contains the global styles (`GlobalStyle.ts`) and the theme for `styled-components` (`theme.ts`).
*   `public/`: Contains static assets like images.

### Styling

The project uses `styled-components` for styling. A consistent theme is defined in `src/styles/theme.ts` and `src/styles/riderTheme.ts`. Global styles are defined in `src/styles/GlobalStyle.ts`.

### Routing

The project uses `react-router-dom` for routing. The routes are defined in `src/App.tsx`. There is a main route that displays the portfolio and a dynamic route for individual project pages.

### Data

The project data, such as the list of projects, is stored in `src/data/projects.ts`. This makes it easy to update the content of the portfolio.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VincentvBruggen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
