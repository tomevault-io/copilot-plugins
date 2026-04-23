## ui

> enableSystem

# Cursor Rules for UI Library Project

# Project Structure

## Atomic Design Structure

- All components must be organized following atomic design principles in `src/components`:
  - `atoms/` - Basic building blocks
    - `button/`
    - `input/`
    - `icon/`
    - `typography/`
  - `molecules/` - Simple combinations of atoms
    - `form-field/`
    - `search-bar/`
    - `card-header/`
  - `organisms/` - Complex combinations of molecules
    - `navigation/`
    - `form/`
    - `card/`
    - `data-table/`
    - `modal/`
    - `sidebar/`

Note: Templates and Pages are not part of the UI library as they are application-specific components that should be implemented in the consuming Next.js application.

## Component Structure

Each component should have its own directory with the following structure:

- `component-name/`
  - `index.ts` (exports)
  - `component-name.tsx` (main component)
  - `component-name.test.tsx` (tests)
  - `component-name.stories.tsx` (Storybook)
  - `component-name.types.ts` (TypeScript types)
  - `component-name.module.css` (if needed)

## Utility Structure

All utility files should use kebab-case naming:

- `hooks/` - Custom React hooks
  - `use-hook-name.ts`
- `utils/` - Utility functions
  - `utils-name.ts`
- `constants/` - Constant values
  - `constants-name.ts`
- `types/` - Type definitions
  - `types-name.ts`
- `styles/` - Global styles
  - `styles-name.css`
- `config/` - Configuration files
  - `config-name.ts`

# Component Rules

## Atomic Design Principles

### Atoms

- Smallest possible components
- Cannot be broken down further
- Examples: buttons, inputs, icons, typography
- Should be highly reusable
- Should have minimal props
- Should be stateless when possible
- Should be the foundation of the design system

### Molecules

- Combinations of atoms
- Form simple, functional units
- Examples: form fields, search bars, card headers
- Should handle simple state
- Should be focused on a single responsibility
- Should be composed of atoms
- Should be reusable across different organisms

### Organisms

- Complex combinations of molecules
- Form distinct sections of an interface
- Examples: navigation bars, forms, cards, data tables
- Can handle complex state
- Can manage their own data fetching
- Should be composed of molecules and atoms
- Should be the highest level of abstraction in the library
- Should be flexible enough to be used in various application contexts
- Should not contain application-specific logic

Note: Templates and Pages are not part of the UI library as they are application-specific components that should be implemented in the consuming Next.js application.

- Use functional components with TypeScript
- Implement proper prop types using TypeScript interfaces
- Use React.memo() for performance optimization when needed
- Follow atomic design principles
- Implement proper error boundaries
- Use proper accessibility attributes (aria-\*)
- Implement proper keyboard navigation
- Use proper semantic HTML elements
- Use kebab-case for component file names and directories
- Use PascalCase for component names in code

# Accessibility Rules

## Core Accessibility Requirements

- All components MUST be fully accessible and follow WCAG 2.1 Level AA guidelines
- Every component must be keyboard navigable and operable
- All interactive elements must have proper focus management
- All form controls must have associated labels
- All images must have meaningful alt text
- All color combinations must meet WCAG contrast requirements
- All components must be screen reader friendly

## ARIA Implementation

- Use semantic HTML elements over ARIA when possible
- When ARIA is needed:
  - Use correct ARIA roles and attributes
  - Ensure all required ARIA attributes are present
  - Validate ARIA attribute values
  - Test with screen readers
- Common ARIA patterns:
  - `aria-label` for elements without visible text
  - `aria-labelledby` for elements with associated labels
  - `aria-describedby` for additional descriptions
  - `aria-expanded` for expandable elements
  - `aria-controls` for elements that control others
  - `aria-live` for dynamic content updates

## Keyboard Navigation

- All interactive elements must be focusable
- Implement proper tab order
- Support keyboard shortcuts where appropriate
- Handle keyboard events (Enter, Space, Escape, etc.)
- Provide visible focus indicators
- Support arrow key navigation for complex components
- Implement skip links for main content

## Screen Reader Support

- Use proper heading hierarchy (h1-h6)
- Provide meaningful text alternatives
- Use proper landmark roles
- Implement proper live regions
- Test with multiple screen readers
- Ensure proper reading order
- Provide proper announcements for dynamic content

## Color and Contrast

- Maintain minimum contrast ratio of 4.5:1 for normal text
- Maintain minimum contrast ratio of 3:1 for large text
- Don't rely solely on color to convey information
- Support high contrast mode
- Test with color blindness simulators
- Provide alternative indicators for color-coded information

## Form Accessibility

- All form controls must have visible labels
- Provide clear error messages
- Support autocomplete where appropriate
- Implement proper validation feedback
- Use proper input types
- Support required field indicators
- Provide clear instructions

## Testing Requirements

- Test with keyboard navigation
- Test with screen readers (NVDA, VoiceOver, JAWS)
- Test with color contrast checkers
- Test with accessibility testing tools (axe, Lighthouse)
- Test with different zoom levels
- Test with different text sizes
- Test with different input methods

## Documentation

- Document all accessibility features
- Add link to a component using this hosting https://sergii-melnykov.github.io/ui like:
/**
 * Avatar component that displays a user's profile picture or fallback.
 * Built on top of Radix UI's Avatar primitive.
 *
 * @url https://sergii-melnykov.github.io/ui/?path=/docs/atoms-avatar--docs
 *
 * @example
 * ```tsx
 * <Avatar>
 *   <AvatarImage src="/path/to/image.jpg" alt="User avatar" />
 *   <AvatarFallback>JD</AvatarFallback>
 * </Avatar>
 * ```
 */
- Provide usage examples
- Document keyboard shortcuts
- Document screen reader behavior
- Document ARIA attributes
- Document focus management
- Document testing procedures

# Styling Rules

- Use Tailwind CSS for styling
- Follow BEM naming convention for custom CSS
- use rem instead px if needed
- Use CSS modules for component-specific styles if needed
- Implement dark mode support
- Use CSS variables for theming
- Follow shadcn/ui and Radix UI styling patterns
- Use proper responsive design patterns
- Use kebab-case for CSS module files and class names

# Testing Rules

- Write unit tests for all components
- Implement integration tests for complex components
- Use React Testing Library
- Maintain minimum 80% test coverage
- Write proper test descriptions
- Test accessibility features
- Test responsive behavior
- Use kebab-case for test file names

# Storybook Rules

- Create stories for all component variants
- Implement proper documentation
- Use proper controls for props
- Implement proper actions
- Use proper decorators
- Implement proper viewport testing
- Use proper accessibility testing
- Use kebab-case for story file names

# TypeScript Rules

- Use strict mode
- No any types
- Use proper type definitions
- Use proper generics
- Use proper utility types
- Use proper type guards
- Use proper type assertions
- Use kebab-case for type definition files

# Performance Rules

- Implement proper code splitting
- Use proper lazy loading
- Implement proper caching
- Use proper memoization
- Implement proper virtualization
- Use proper image optimization
- Implement proper bundle size optimization

# Documentation Rules

- Write proper JSDoc comments
- Document all props
- Document all events
- Document all methods
- Document all types
- Document all stories
- Document all tests
- Use kebab-case for documentation files

# Git Rules

- Use conventional commits
- Write proper commit messages
- Use proper branch naming
- Use proper PR descriptions
- Use proper issue templates
- Use proper release notes
- Use proper versioning
- Use kebab-case for branch names and feature flags

# Build Rules

- Use proper Vite configuration
- Use proper Rollup configuration
- Use proper Next.js configuration
- Use proper TypeScript configuration
- Use proper ESLint configuration
- Use proper Prettier configuration
- Use proper PostCSS configuration
- Use kebab-case for configuration files

# Dependencies Rules

- Keep dependencies up to date
- Use proper versioning
- Use proper peer dependencies
- Use proper dev dependencies
- Use proper optional dependencies
- Use proper resolutions
- Use proper engines

# Security Rules

- Implement proper input validation
- Use proper sanitization
- Implement proper XSS protection
- Use proper CSRF protection
- Implement proper rate limiting
- Use proper authentication
- Use proper authorization

# Internationalization Rules

- Use proper i18n setup
- Implement proper translations
- Use proper date formatting
- Use proper number formatting
- Use proper currency formatting
- Use proper pluralization
- Use proper RTL support
- Use kebab-case for translation files and keys

# shadcn/ui Rules

## Component Integration

- Use shadcn/ui as the primary component library foundation
- Install components using the CLI: `npx shadcn-ui@latest add [component-name]`
- Keep components in sync with the latest shadcn/ui version
- Customize components through the `components.json` configuration
- Follow the shadcn/ui component structure:
  - `components/ui/` - Base shadcn components
- import { cn } from "@/utils"

## Component Customization

- Extend shadcn components through composition rather than direct modification
- Use the `cn()` utility for conditional class merging
- Maintain consistent prop patterns across custom components
- Document any deviations from standard shadcn patterns
- Use the `asChild` pattern for polymorphic components
- Implement proper TypeScript types for all customizations

## Theme Configuration

### Base Setup

- Use the `tailwind.config.js` for theme configuration
- Define color palette in `tailwind.config.js`:
  ```js
  theme: {
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        // ... other color definitions
      }
    }
  }
  ```

### CSS Variables

- Define CSS variables in `globals.css`:
  ```css
  @layer base {
    :root {
      --background: 0 0% 100%;
      --foreground: 222.2 84% 4.9%;
      --card: 0 0% 100%;
      --card-foreground: 222.2 84% 4.9%;
      --popover: 0 0% 100%;
      --popover-foreground: 222.2 84% 4.9%;
      --primary: 222.2 47.4% 11.2%;
      --primary-foreground: 210 40% 98%;
      /* ... other variables */
    }
 
    .dark {
      --background: 222.2 84% 4.9%;
      --foreground: 210 40% 98%;
      /* ... dark mode variables */
    }
  }
  ```

## Theme Switching

### Implementation

- Use `next-themes` for theme management
- Implement theme provider in root layout:
  ```tsx
  import { ThemeProvider } from "next-themes"
  
  export default function RootLayout({ children }) {
    return (
      <html lang="en" suppressHydrationWarning>
        <body>
          <ThemeProvider
            attribute="class"
            defaultTheme="system"
            enableSystem
            disableTransitionOnChange
          >
            {children}
          </ThemeProvider>
        </body>
      </html>
    )
  }
  ```

### Theme Switching Component

- Create a reusable theme toggle component:
  ```tsx
  import { useTheme } from "next-themes"
  import { Button } from "@/components/ui/button"
  import { Moon, Sun } from "lucide-react"
  
  export function ThemeToggle() {
    const { theme, setTheme } = useTheme()
  
    return (
      <Button
        variant="ghost"
        size="icon"
        onClick={() => setTheme(theme === "light" ? "dark" : "light")}
      >
        <Sun className="h-5 w-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
        <Moon className="absolute h-5 w-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
        <span className="sr-only">Toggle theme</span>
      </Button>
    )
  }
  ```

## Best Practices

### Component Usage

- Use shadcn components as building blocks
- Maintain consistent spacing using the theme's spacing scale
- Use semantic color tokens instead of direct color values
- Implement proper dark mode support for all custom components
- Use the `cn()` utility for conditional styling
- Follow the shadcn component API patterns

### Theme Customization

- Extend the theme through `tailwind.config.js`
- Use CSS variables for dynamic theming
- Maintain consistent naming conventions
- Document all custom theme extensions
- Test components in both light and dark modes
- Ensure proper contrast ratios in both themes

### Performance

- Use CSS variables for theme switching
- Implement proper transition animations
- Avoid theme-dependent layout shifts
- Use proper color contrast in both themes
- Optimize theme switching performance
- Implement proper hydration handling

### Accessibility

- Ensure proper color contrast in both themes
- Maintain consistent focus indicators
- Test with screen readers in both themes
- Implement proper ARIA attributes
- Support system theme preferences
- Provide manual theme override options

DO NOT EXPLAIN WHAT DO YOU DO IF NO ONE ASK YOU

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sergii-melnykov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
