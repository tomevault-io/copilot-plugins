## t-rex-simplified

> Apply this rule whenever modifying shared components, dependencies, or logic to ensure no regressions or broken functionality occur elsewhere in the front-end codebase.


# General Code Style & Formatting
- Use functional and declarative programming patterns; avoid classes.  
- Prefer iteration and modularization over code duplication.  
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).  
- Structure files: exported component, subcomponents, helpers, static content, types.  
- Follow Expo's official documentation for setting up and configuring projects.  
- Always ensure that any code change does not break existing dependencies or functionality in the front-end codebase.  
- When modifying a component or dependency, check for and validate all other components that rely on the same dependency or logic to maintain consistency and prevent regressions.  

**Description:** Apply this rule whenever modifying shared components, dependencies, or logic to ensure no regressions or broken functionality occur elsewhere in the front-end codebase.  

# Naming Conventions
- Use lowercase with dashes for directories (e.g., components/auth-wizard).  
- Favor named exports for components.  

# TypeScript Best Practices
- Use TypeScript for all code; prefer interfaces over types.  
- Avoid any and enums; use explicit types and maps instead.  
- Use functional components with TypeScript interfaces.  
- Enable strict mode in TypeScript for better type safety.  

# Syntax & Formatting
- Use the function keyword for pure functions.  
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.  
- Use declarative JSX.  
- Use Prettier for consistent code formatting.  

# Styling & UI
- Use Expo's built-in components for common UI patterns and layouts.  
- Implement responsive design with Flexbox and useWindowDimensions.  
- Use styled-components or Tailwind CSS for styling.  
- Implement dark mode support using Expo's useColorScheme.  
- Ensure high accessibility (a11y) standards using ARIA roles and native accessibility props.  
- Use react-native-reanimated and react-native-gesture-handler for performant animations and gestures.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Stokoloko89) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
