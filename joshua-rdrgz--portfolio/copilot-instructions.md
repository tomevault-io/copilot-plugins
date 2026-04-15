## portfolio

> - TypeScript (Latest stable version)

# React & TypeScript Development Guidelines

## Core Technologies

- TypeScript (Latest stable version)
- React (Latest stable version)
- Next.js App Router (Latest stable version)
- Node.js (Latest stable version)
- Shadcn UI
- Tailwind CSS

## 1. Naming Conventions

**Scope**: `*.{ts,tsx,js,jsx}`

- **Components**: Use PascalCase
  ```typescript
  function UserProfile() {}
  function NavigationBar() {}
  ```
- **Variables & Functions**: Use camelCase
  ```typescript
  const userData = {};
  function handleSubmit() {}
  ```

## 2. TypeScript Guidelines

**Scope**: `*.{ts,tsx}`

- Use strict typing throughout the codebase
- Prefer interfaces over types for object definitions

  ```typescript
  // Preferred
  interface UserData {
    id: string;
    name: string;
  }

  // Avoid when possible
  type UserData = {
    id: string;
    name: string;
  };
  ```

- Implement generics for reusable components/functions
- Avoid using `any` type - use proper type definitions
- Enable strict TypeScript configuration

## 3. UI and Styling

**Scope**: `*.{ts,tsx,js,jsx}`

### Tailwind CSS

- Utilize Tailwind utility classes for styling
- Follow responsive design principles
- Maintain consistent spacing and layout patterns

### Shadcn UI

- Follow component guidelines and best practices
- Maintain consistent component usage patterns
- Utilize built-in accessibility features

### Accessibility

- Ensure proper ARIA attributes
- Maintain semantic HTML structure
- Support keyboard navigation
- Provide appropriate color contrast

## 4. Performance Optimization

**Scope**: `*.{ts,tsx,js,jsx}`

### React Optimization

- Implement React.memo for pure components
- Use useMemo and useCallback hooks appropriately
- Avoid unnecessary re-renders
- Implement code splitting and lazy loading

### General Optimization

- Optimize images and assets
- Use efficient data structures
- Implement proper caching strategies
- Monitor and optimize bundle size

## 5. Code Quality Standards

### Best Practices

- Write clear, self-documenting code
- Maintain consistent code formatting
- Follow DRY (Don't Repeat Yourself) principles
- Implement proper error handling

### Component Structure

- Keep components focused and single-responsibility
- Implement proper prop typing
- Maintain clear component hierarchy
- Document complex logic with comments

## 6. Development Approach

- Write comprehensive, production-ready code
- Implement complete feature requirements
- Focus on maintainability and readability
- Provide thorough documentation when needed
- Consider edge cases and error states

## 7. Testing Considerations

- Write unit tests for components
- Implement integration tests for features
- Test for accessibility compliance
- Ensure responsive design testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshua-rdrgz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
