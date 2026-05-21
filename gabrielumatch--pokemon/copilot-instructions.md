## expo

> - We are using Expo SDK 52 and Expo Router 4

# Expo Project Rules

## Project Configuration
- We are using Expo SDK 52 and Expo Router 4
- Always check the [Expo Documentation](mdc:https:/docs.expo.dev) when implementing new features
- Always check the [Expo Router Documentation](mdc:https:/expo.github.io/router/docs) for routing-specific implementations
- Async Storage
- Responsive

## Styling Guidelines
- Always use styled-components for styling
- Create styled components in separate files with `.styles.ts` extension
- Keep global theme configuration in a centralized theme file

```typescript
// Example of styled components implementation
import styled from 'styled-components/native';

export const Container = styled.View`
  flex: 1;
  background-color: ${props => props.theme.colors.background};
`;

export const Title = styled.Text`
  font-size: 18px;
  color: ${props => props.theme.colors.text};
`;
```

## File Structure
- Screen components should be in `.tsx` files
- Business logic, hooks, and handlers should be in `.ts` files
- Organize related files in feature folders
- Folder for Types

## Routing Guidelines
- Use Expo Router for navigation
- Implement URL-based navigation with file-based routing
- Place layouts in `_layout.tsx` files
- Use URL params for dynamic routes

## Component Guidelines
- Split screens into smaller reusable components
- Use TypeScript interfaces for component props
- Implement proper loading and error states

## State Management
- Use React hooks for local state
- Implement custom hooks for reusable logic
- Keep business logic separate from UI components

## Performance Guidelines
- Implement proper memoization for expensive computations
- Use `React.memo()` for components that re-render often
- Avoid unnecessary re-renders

## Expo Best Practices
- Use Expo modules over native modules when available
- Implement proper permissions handling
- Test on both iOS and Android regularly
- Follow Expo's security guidelines

## TypeScript Usage
- Use strict typing for all components and functions
- Define interfaces for all component props
- Use type guards when necessary

---
> Source: [gabrielumatch/pokemon](https://github.com/gabrielumatch/pokemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
