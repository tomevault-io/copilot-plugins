## shivlink

> This is a React/Vite website for comparing telecom plans (specifically Rogers and Telus plans in Canada). The site includes:

# shivlink - Claude Code Guidelines

## Project Overview
This is a React/Vite website for comparing telecom plans (specifically Rogers and Telus plans in Canada). The site includes:
- Hero section with call-to-action
- Rogers and Telus plan comparison pages
- About page
- Contact page
- Responsive design with Tailwind CSS
- Dark/light mode support
- Smooth animations with Framer Motion

## Tech Stack
- **Framework**: React 18 with React Router DOM v6
- **Build Tool**: Vite
- **Styling**: Tailwind CSS 3.4
- **Animations**: Framer Motion
- **Icons**: Lucide React
- **Language**: TypeScript
- **Linting**: ESLint with React plugins
- **State Management**: React hooks (useState, useEffect, etc.)

## Project Structure
```
src/
├── App.tsx          # Main app component with routing
├── main.tsx         # Entry point
├── index.css        # Global CSS
├── vite-env.d.ts    # Vite TypeScript definitions
├── components/      # Reusable components
│   ├── Navbar.tsx   # Navigation bar
│   ├── Hero.tsx     # Hero section
│   ├── PlanCard.tsx # Plan card component
│   └── Footer.tsx   # Footer
├── pages/           # Page components
│   ├── RogersPlans.tsx
│   ├── TelusPlans.tsx
│   ├── About.tsx
│   └── Contact.tsx
└── (other asset files)
```

## Development Setup
1. Install dependencies: `npm install`
2. Start development server: `npm run dev`
3. Build for production: `npm run build`
4. Preview production build: `npm run preview`
5. Lint code: `npm run lint`

## Coding Conventions
- **File Extensions**: Use `.tsx` for React components, `.ts` for utility functions
- **Component Naming**: PascalCase (e.g., `PlanCard.tsx`)
- **Function Naming**: camelCase
- **TypeScript**: Prefer interfaces over types for object shapes
- **Styling**: Tailwind utility-first approach
- **Imports**:
  - React imports first
  - Then third-party libraries
  - Then local components
  - Then local styles/assets
- **Export**: Default export for pages and main components, named exports for utilities
- **Formatting**: Use Prettier (configured via ESLint)

## Common Tasks
- **Adding a new page**:
  1. Create component in `src/pages/`
  2. Add route in `src/App.tsx`
  3. Link from Navbar if needed
- **Adding a new component**:
  1. Create in `src/components/`
  2. Import and use where needed
  3. Follow existing patterns for props and styling
- **Updating styles**:
  - Use Tailwind utility classes
  - For custom CSS, add to `src/index.css`
  - Maintain responsive design principles
- **State Management**:
  - Use React hooks (useState, useEffect, useContext)
  - For complex state, consider lifting state up
  - Avoid prop drilling by using Context when appropriate

## Best Practices
- Keep components small and focused
- Use TypeScript strictly for type safety
- Follow accessibility guidelines (a11y)
- Optimize images and assets
- Use lazy loading for route-based code splitting when appropriate
- Test changes across different screen sizes
- Maintain consistent spacing and alignment using Tailwind's spacing scale

## Available Scripts
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build
- `npm run lint` - Run ESLint

## Environment Variables
Currently no environment variables are used, but if needed:
- Store in `.env` file (not committed)
- Prefix with `VITE_` for Vite to expose them to client code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sharathannaiah)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sharathannaiah)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
