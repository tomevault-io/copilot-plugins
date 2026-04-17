## samchang

> This is a Gatsby-based portfolio website using TypeScript, Emotion for styling, and various Gatsby plugins. The project follows a component-based architecture with proper TypeScript typing.

# Cursor Rules for Sam Chang's Gatsby Portfolio

## Project Overview
This is a Gatsby-based portfolio website using TypeScript, Emotion for styling, and various Gatsby plugins. The project follows a component-based architecture with proper TypeScript typing.

## Technology Stack
- **Framework**: Gatsby 5.x
- **Language**: TypeScript 4.x
- **Styling**: Emotion (@emotion/react, @emotion/styled)
- **Testing**: Jest + React Testing Library
- **Linting**: TSLint + Prettier
- **Package Manager**: Yarn (preferred) or npm

## Code Style & Conventions

### TypeScript
- Use strict TypeScript configuration
- Always provide proper type annotations for function parameters and return types
- Use interfaces for component props
- Avoid `any` type - use proper typing instead
- Use `React.FC` or explicit function component typing

### React Components
- Use functional components with hooks
- Follow the existing component structure in `src/components/`
- Each component should have its own directory with an `index.tsx` file
- Use proper prop interfaces for all components
- Implement proper error boundaries where needed

### Styling with Emotion
- Use `@emotion/styled` for styled components
- Use `@emotion/react` for the `jsx` pragma
- Follow the existing theme structure in `src/components/Theme/`
- Use CSS custom properties (variables) for theming
- Implement responsive design using the `mq` breakpoints from `src/styles/styles`

### File Structure
```
src/
├── components/          # Reusable UI components
│   ├── ComponentName/
│   │   └── index.tsx   # Main component file
├── pages/              # Gatsby pages
├── templates/          # Gatsby templates
├── styles/             # Global styles and utilities
├── images/             # Static images
└── projects/           # Project data
```

## Specific Rules

### 1. Component Props Interface
Always define interfaces for component props:
```typescript
interface ComponentProps {
  children: React.ReactNode;
  className?: string;
  // other props...
}

export default function Component({ children, className }: ComponentProps) {
  // component implementation
}
```

### 2. Styled Components with Props
When using styled components with props, define the prop interface:
```typescript
interface StyledComponentProps {
  showHeaderShadow?: boolean;
  // other props...
}

const StyledComponent = styled.div<StyledComponentProps>`
  box-shadow: ${props =>
    props.showHeaderShadow ? "var(--shadow) 0px 1px 0px" : "none"};
`;
```

### 3. Gatsby Specific Patterns
- Use Gatsby's built-in image optimization with `gatsby-plugin-image`
- Follow the existing data sourcing patterns for projects and music
- Use Gatsby's routing system for navigation
- Implement proper SEO with `gatsby-plugin-react-helmet`

### 4. Performance Considerations
- Use `throttle-debounce` for scroll events (already implemented)
- Implement proper memoization with `useMemo` and `useCallback`
- Use Gatsby's image optimization features
- Follow the existing performance patterns in the codebase

### 5. Testing
- Write tests for all components using Jest and React Testing Library
- Follow the existing test patterns in the `tests/` directory
- Use proper test data and mocking strategies

### 6. Accessibility
- Implement proper ARIA labels and roles
- Use semantic HTML elements
- Ensure keyboard navigation works properly
- Follow WCAG guidelines

### 7. Error Handling
- Implement proper error boundaries
- Use TypeScript's strict mode to catch type errors
- Handle edge cases in data fetching and user interactions

## Common Issues to Fix

### TypeScript Issues
- Add proper type definitions for external libraries (e.g., `@types/throttle-debounce`)
- Fix implicit `any` types by providing proper interfaces
- Ensure all component props are properly typed

### Styling Issues
- Use the existing theme system for consistent spacing and colors
- Follow the responsive design patterns with `mq` breakpoints
- Use CSS custom properties for theming

### Performance Issues
- Implement proper cleanup in useEffect hooks
- Use throttling for scroll events (already implemented)
- Optimize images using Gatsby's image plugins

## Development Workflow
1. Use `yarn` for package management
2. Run `yarn type-check` to verify TypeScript compilation
3. Run `yarn lint` to check for linting issues
4. Run `yarn test` to run the test suite
5. Use Prettier for code formatting (configured in lint-staged)

## Dependencies
- Always check existing dependencies before adding new ones
- Prefer Gatsby plugins over generic React libraries when possible
- Use the existing emotion setup for styling
- Follow the existing testing setup with Jest and React Testing Library

## Code Quality
- Write self-documenting code with clear variable and function names
- Add JSDoc comments for complex functions
- Follow the existing code patterns in the codebase
- Ensure all code passes the linter and type checker
- Write meaningful commit messages

## Security
- Never commit sensitive data or API keys
- Use environment variables for configuration
- Follow security best practices for user input handling
- Keep dependencies updated

## Performance
- Optimize bundle size by using code splitting
- Implement proper caching strategies
- Use Gatsby's built-in performance optimizations
- Monitor Core Web Vitals

Remember to always check the existing codebase patterns and follow the established conventions when making changes or adding new features. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skrapkam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
