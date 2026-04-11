## udemy-indecision-app

> Modernized React 18 application upgraded from legacy Udemy course. Uses functional components with hooks, webpack 5, Babel 7, Jest 29, and TypeScript support.

# AGENTS.md - Agentic Coding Guidelines for Indecision App

## Project Overview
Modernized React 18 application upgraded from legacy Udemy course. Uses functional components with hooks, webpack 5, Babel 7, Jest 29, and TypeScript support.

## Build Commands

```bash
# Development
npm run build:dev          # Webpack 5 development build with dev server
npm run dev-server         # Same as build:dev

# Production
npm run build:prod         # Webpack 5 production build

# Testing
npm test                   # Run all tests with Jest 29
npm test -- --watch       # Watch mode for tests
npm test -- -t "test name"    # Run single test by name

# Server
npm start                  # Run Express server (port from env.PORT or 3000)
```

**Running a Single Test:**
```bash
# By test name pattern
npm test -- -t "handleAddOption"

# Watch + single test
npm test -- --watch -t "handleAddOption"
```

---

## Code Style Guidelines

### General Principles
- Follow modern React patterns with functional components and hooks
- TypeScript support added (optional migration path)
- Consistent with Vercel React best practices
- Avoid adding unnecessary dependencies

### File Organization
```
src/
├── app.js                 # Entry point (React 18 createRoot)
├── components/            # React components (functional with hooks)
│   ├── IndecisionApp.js
│   ├── AddOption.js
│   └── ...
├── styles/                # SCSS files
│   ├── styles.scss        # Main entry
│   ├── base/              # Settings, base styles
│   └── components/        # Component-specific styles
├── public/                # Static assets
│   └── index.html         # HTML template
└── playground/            # Scratch files (ignore)
```

### Imports
- React imports first, then third-party, then relative
- Order: React → external → components → styles
```javascript
import React, { useState, useEffect } from 'react';
import AddOption from './AddOption';
import './styles/styles.scss';
```

### Naming Conventions
- **Components**: PascalCase (`IndecisionApp.js`, `AddOption.js`)
- **Files**: Match component name exactly
- **Functions**: Use functional components with hooks
- **Methods**: camelCase
```javascript
export default function IndecisionApp() {
  const [options, setOptions] = useState([]);
  // ...
}
```

### Component Patterns
- Use functional components with hooks (useState, useEffect)
- Custom hooks for complex logic
- Event handlers as inline arrow functions or useCallback
- Conditional rendering with && or ternary operators
```javascript
export default function MyComponent() {
  const [value, setValue] = useState('');
  
  const handleClick = () => {
    // ...
  };
  
  return (
    <div>
      {value && <p>{value}</p>}
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}
```

### Props and State
- Access via function parameters and hooks
- Use useState/useReducer for state management
- Use useMemo/useCallback for performance optimization
- Never mutate state directly

### CSS Class Names
- Use BEM-ish naming: `widget-header`, `widget-header__title`, `button--link`
- Component styles in `src/styles/components/_<component>.scss`

### Error Handling
- Use try/catch in useEffect or event handlers
- Error boundaries for catching render errors (consider implementing)
- Console.log for debugging (remove in production)

---

## Testing with Jest + React Testing Library

### Setup
- Jest configured in `jest.config.json`
- Uses @testing-library/react and @testing-library/jest-dom
- JSDOM test environment

### Test File Pattern
- Tests live next to components: `ComponentName.test.js`
- Or in `__tests__` folder at same level

### Running Tests
```bash
npm test                          # All tests
npm test -- -t "pattern"         # Tests matching name
npm test -- --watch              # Watch mode
```

### Testing Library Usage
```javascript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// Render component
render(<MyComponent />);

// Find elements
screen.getByRole('button', { name: /submit/i });
screen.getByLabelText(/username/i);

// Simulate events
userEvent.click(screen.getByRole('button'));
userEvent.type(screen.getByRole('textbox'), 'hello');

// Assertions
expect(screen.getByText(/hello/i)).toBeInTheDocument();
```

---

## webpack Configuration
- Entry: `src/app.js`
- Output: `public/bundle.js`
- Loaders: 
  - babel-loader (JS/JSX)
  - style-loader + css-loader + sass-loader (SCSS)
  - style-loader + css-loader (CSS)
- Plugins: HtmlWebpackPlugin
- DevServer: Hot module replacement, port 8080

---

## Dependencies (Current Stack)
- React 18.2.0 / React DOM 18.2.0
- React Router 6.4.0
- webpack 5.75.0
- Babel 7.x (@babel/core, @babel/preset-env, @babel/preset-react)
- Jest 29.x with React Testing Library
- Sass 1.55.0 with sass-loader
- Normalize.css 8.0.1
- React Modal 3.15.1
- UUID 9.0.0
- Validator 13.7.0
- Express 4.18.0 (server)
- TypeScript support (tsconfig.json included)

---

## Working in This Codebase

1. **DO**: Follow modern React patterns with functional components and hooks
2. **DO**: Use useState/useEffect for state and side effects
3. **DO**: Test changes with `npm run build:dev` before committing
4. **DO**: Run `npm test` before submitting
5. **DO**: Consider migrating to TypeScript for new components
6. **DON'T**: Use class components (unless maintaining existing ones temporarily)
7. **DON'T**: Add unnecessary dependencies
8. **DON'T**: Mutate state directly
9. **DON'T**: Use index as key in lists when stable IDs are available

---

## Common Tasks

### Adding a new component
1. Create `src/components/NewComponent.js`
2. Export default functional component with hooks
3. Add corresponding SCSS in `src/styles/components/`
4. Import in parent component

### Running the app locally
```bash
npm run dev-server   # Then open http://localhost:8080
```

### Production build
```bash
npm run build:prod   # Outputs to public/bundle.js
```

### Converting class components to functional
1. Replace class with function
2. Convert `this.state` to `useState()` calls
3. Convert lifecycle methods to `useEffect()`
4. Convert methods to useCallback or inline functions
5. Remove `this.` references

---

**Last Updated**: 2026-03-25

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dsabalete)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dsabalete)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
