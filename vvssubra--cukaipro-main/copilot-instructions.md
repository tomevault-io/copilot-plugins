## cukaipro-main

> CukaiPro is a modern tax management platform for Malaysian businesses built with React, Vite, and Tailwind CSS.

# CukaiPro Project - Cursor AI Rules

## Project Overview
CukaiPro is a modern tax management platform for Malaysian businesses built with React, Vite, and Tailwind CSS.

## Code Style & Patterns

### React Best Practices
- Use functional components with hooks
- Prefer named exports for components
- Use custom hooks for reusable logic
- Implement proper error boundaries
- Use React.lazy() and Suspense for code splitting

### State Management
- Use Context API for global state (auth, theme)
- Use local state (useState) for component-specific state
- Use useReducer for complex state logic

### File Structure
```
src/
├── components/
│   ├── Common/       # Reusable UI components
│   └── Auth/         # Authentication components
├── pages/            # Page-level components
├── hooks/            # Custom React hooks
├── utils/            # Utility functions
├── context/          # Context providers
├── types/            # TypeScript definitions
└── styles/           # Global styles
```

### Naming Conventions
- Components: PascalCase (e.g., `LoginForm.jsx`)
- Hooks: camelCase with 'use' prefix (e.g., `useAuth.js`)
- Utils: camelCase (e.g., `formatCurrency.js`)
- Constants: UPPER_SNAKE_CASE (e.g., `API_BASE_URL`)

### Component Structure
```jsx
import { useState, useEffect } from 'react';
import PropTypes from 'prop-types';

function ComponentName({ prop1, prop2 }) {
  // Hooks
  const [state, setState] = useState(null);
  
  // Effects
  useEffect(() => {
    // Effect logic
  }, []);
  
  // Event handlers
  const handleEvent = () => {
    // Handler logic
  };
  
  // Render
  return (
    <div>
      {/* JSX */}
    </div>
  );
}

ComponentName.propTypes = {
  prop1: PropTypes.string.isRequired,
  prop2: PropTypes.number,
};

export default ComponentName;
```

### API Integration
- Use axios for HTTP requests
- Centralize API calls in service files
- Use custom hooks for data fetching
- Implement proper error handling
- Show loading states

### Form Handling
- Use react-hook-form for forms
- Use Zod for validation schemas
- Provide clear error messages
- Implement debouncing for API calls

### Testing
- Write tests for critical components
- Use Vitest and React Testing Library
- Test user interactions, not implementation
- Mock external dependencies

### Performance
- Use React.memo() for expensive components
- Implement useMemo and useCallback where appropriate
- Lazy load routes and heavy components
- Optimize images and assets

### Security
- Never commit API keys or secrets
- Sanitize user inputs
- Use HTTPS for API calls
- Implement proper authentication
- Store sensitive data securely

### Styling
- Use Tailwind CSS utility classes
- Follow mobile-first approach
- Support dark mode with dark: prefix
- Keep consistent spacing and colors
- Use semantic color names from theme

### Error Handling
- Use ErrorBoundary for component errors
- Log errors to console in development
- Show user-friendly error messages
- Provide error recovery actions

## Code Quality
- Run prettier before committing
- Fix all ESLint warnings
- Write descriptive commit messages
- Document complex logic
- Keep functions small and focused

## Malaysian Tax Context
- Support SST, EA forms, LHDN requirements
- Use RM currency format
- Support Malaysian date formats
- Implement LHDN e-Invoice compatibility
- Use Malaysian fiscal year (Jan-Dec)

## When Generating Code
1. Follow the existing code patterns in the project
2. Use TypeScript types when possible (even in .jsx files via JSDoc)
3. Implement proper loading and error states
4. Add accessibility attributes (ARIA)
5. Support keyboard navigation
6. Test on mobile viewports
7. Consider performance implications
8. Add comments for complex business logic

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vvssubra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
