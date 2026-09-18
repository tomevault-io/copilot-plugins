## project-structure

> This rule defines the comprehensive file organization and project structure for zen-flow-ui. Follow this structure when creating new files, organizing components, or refactoring the codebase.

# Zen Flow UI Project Structure

## Overview

This rule defines the comprehensive file organization and project structure for zen-flow-ui. Follow this structure when creating new files, organizing components, or refactoring the codebase.

## Root Directory Structure

```
zen-flow-ui/
├── .cursor/                    # Cursor IDE rules and configurations
│   └── rules/                  # Cursor rules for project guidelines
├── .github/                    # GitHub workflows and templates
│   ├── ISSUE_TEMPLATE/         # Issue templates
│   └── workflows/              # CI/CD workflows
├── cli/                        # CLI tools and utilities
│   ├── index.cjs              # Main CLI entry point
│   ├── commands/              # Individual CLI commands
│   ├── templates/             # Project templates
│   └── utils/                 # CLI utility functions
├── docs/                      # Documentation files
│   ├── GETTING_STARTED.md     # Quick start guide
│   ├── CONTRIBUTING.md        # Contribution guidelines
│   ├── DEPLOYMENT.md          # Deployment instructions
│   ├── MIGRATION_GUIDES/      # Version migration guides
│   └── API_REFERENCE/         # Component API documentation
├── examples/                  # Usage examples and demos
│   ├── App.tsx               # Demo application
│   ├── components/           # Example component usage
│   └── templates/            # Starter templates
├── scripts/                  # Build and automation scripts
│   ├── build.js             # Build script
│   ├── setup-tailwind.js    # Tailwind configuration helper
│   ├── generate-exports.js  # Auto-generate export files
│   └── migrate.js           # Migration utilities
├── src/                     # Main source code
│   ├── components/          # React components
│   ├── hooks/              # Custom React hooks
│   ├── lib/                # Utility libraries and core functions
│   ├── styles/             # Global styles and CSS
│   └── types/              # TypeScript type definitions
├── tests/                  # Test utilities and global test setup
│   ├── setup.ts           # Test environment setup
│   ├── utils.ts           # Test utilities
│   └── mocks/             # Mock implementations
├── .eslintrc.json         # ESLint configuration
├── .gitignore            # Git ignore patterns
├── jest.config.js        # Jest testing configuration
├── package.json          # Package configuration and dependencies
├── postcss.config.js     # PostCSS configuration
├── README.md             # Project overview and basic documentation
├── rollup.config.js      # Rollup build configuration
├── tailwind.config.js    # Tailwind CSS configuration
└── tsconfig.json         # TypeScript configuration
```

## Source Code Structure

### Components Directory (`src/components/`)

```
src/components/
├── ui/                           # Core UI components
│   ├── Accordion/               # Accordion component group
│   │   ├── index.ts            # Export barrel
│   │   ├── Accordion.tsx       # Main component
│   │   ├── AccordionItem.tsx   # Sub-component
│   │   ├── types.ts           # Type definitions
│   │   └── Accordion.test.tsx  # Unit tests
│   ├── Alert/                  # Alert component group
│   ├── Button/                 # Button component group
│   │   ├── index.ts           # Export barrel
│   │   ├── Button.tsx         # Main button component
│   │   ├── ButtonGroup.tsx    # Button group component
│   │   ├── types.ts          # Button-specific types
│   │   └── Button.test.tsx    # Button tests
│   ├── Card/                  # Card component group
│   ├── DataTable/             # Data table component group
│   ├── Dialog/                # Dialog component group
│   ├── Form/                  # Form component group
│   │   ├── index.ts          # Export barrel
│   │   ├── Input.tsx         # Input component
│   │   ├── Select.tsx        # Select component
│   │   ├── Textarea.tsx      # Textarea component
│   │   ├── RadioGroup.tsx    # Radio group component
│   │   ├── Toggle.tsx        # Toggle/switch component
│   │   ├── Slider.tsx        # Slider component
│   │   └── types.ts          # Form-related types
│   ├── Layout/               # Layout components
│   │   ├── index.ts         # Export barrel
│   │   ├── Container.tsx    # Container component
│   │   ├── Grid.tsx         # Grid system
│   │   ├── Stack.tsx        # Stack layout
│   │   └── Flex.tsx         # Flex layout
│   ├── Navigation/          # Navigation components
│   │   ├── index.ts        # Export barrel
│   │   ├── Breadcrumb.tsx  # Breadcrumb navigation
│   │   ├── Tabs.tsx        # Tab navigation
│   │   ├── Command.tsx     # Command palette
│   │   └── types.ts        # Navigation types
│   ├── Overlay/            # Overlay components
│   │   ├── index.ts       # Export barrel
│   │   ├── Modal.tsx      # Modal component
│   │   ├── Popover.tsx    # Popover component
│   │   ├── Tooltip.tsx    # Tooltip component
│   │   └── types.ts       # Overlay types
│   ├── Feedback/          # Feedback components
│   │   ├── index.ts      # Export barrel
│   │   ├── Progress.tsx  # Progress indicators
│   │   ├── Notification.tsx # Toast notifications
│   │   ├── Alert.tsx     # Alert messages
│   │   └── types.ts      # Feedback types
│   └── Animation/         # Animation components
│       ├── index.ts      # Export barrel
│       ├── ZenTransition.tsx # GSAP transition wrapper
│       ├── FadeIn.tsx    # Fade in animation
│       ├── SlideUp.tsx   # Slide up animation
│       └── types.ts      # Animation types
└── providers/            # Context providers
    ├── index.ts         # Export barrel
    ├── ThemeProvider.tsx # Theme context
    ├── AnimationProvider.tsx # Animation context
    └── ToastProvider.tsx # Toast notification context
```

### Hooks Directory (`src/hooks/`)

```
src/hooks/
├── index.ts              # Export barrel for all hooks
├── animation/            # Animation-related hooks
│   ├── useZenAnimation.ts # Main GSAP animation hook
│   ├── useReducedMotion.ts # Reduced motion detection
│   ├── useStagger.ts     # Staggered animations
│   └── useScrollTrigger.ts # Scroll-triggered animations
├── form/                 # Form-related hooks
│   ├── useFormValidation.ts # Form validation logic
│   ├── useInputState.ts  # Input state management
│   └── useFormContext.ts # Form context access
├── layout/               # Layout-related hooks
│   ├── useBreakpoint.ts  # Responsive breakpoint detection
│   ├── useViewportSize.ts # Viewport size tracking
│   └── useResizeObserver.ts # Element resize observation
├── interaction/          # User interaction hooks
│   ├── useHover.ts       # Hover state management
│   ├── useFocus.ts       # Focus state management
│   ├── useKeyboard.ts    # Keyboard event handling
│   └── useClickOutside.ts # Click outside detection
└── utility/              # General utility hooks
    ├── useLocalStorage.ts # Local storage integration
    ├── useDebounce.ts    # Value debouncing
    ├── useThrottle.ts    # Value throttling
    └── useId.ts          # Unique ID generation
```

### Library Directory (`src/lib/`)

```
src/lib/
├── index.ts                # Main library exports
├── animations/             # Animation system
│   ├── index.ts           # Animation exports
│   ├── core.ts            # Core GSAP setup and configuration
│   ├── presets.ts         # Pre-defined animation presets
│   ├── timing.ts          # Timing and easing constants
│   ├── interactions.ts    # Interactive animation helpers
│   ├── transitions.ts     # Page transition animations
│   └── utils.ts           # Animation utility functions
├── theme/                 # Theme system
│   ├── index.ts          # Theme exports
│   ├── colors.ts         # Color palette definitions
│   ├── typography.ts     # Typography scale and settings
│   ├── spacing.ts        # Spacing scale and utilities
│   ├── shadows.ts        # Shadow and elevation settings
│   ├── borders.ts        # Border and radius settings
│   └── constants.ts      # Theme constant values
├── utils/                # Utility functions
│   ├── index.ts         # Utility exports
│   ├── cn.ts            # Class name utility (clsx + tailwind-merge)
│   ├── accessibility.ts # Accessibility helpers
│   ├── dom.ts           # DOM manipulation utilities
│   ├── formatting.ts    # Data formatting utilities
│   ├── validation.ts    # Input validation functions
│   └── colors.ts        # Color manipulation utilities
├── context/             # React context utilities
│   ├── index.ts        # Context exports
│   ├── createContext.ts # Context creation helpers
│   └── providers.ts    # Context provider utilities
└── constants/          # Application constants
    ├── index.ts       # Constant exports
    ├── breakpoints.ts # Responsive breakpoints
    ├── zIndex.ts      # Z-index scale
    └── keys.ts        # Keyboard key constants
```

### Styles Directory (`src/styles/`)

```
src/styles/
├── globals.css           # Global styles and CSS custom properties
├── components/           # Component-specific styles (if needed)
│   ├── accordion.css    # Accordion-specific styles
│   ├── button.css       # Button-specific styles
│   └── animations.css   # Animation keyframes and utilities
├── utilities/           # Utility CSS classes
│   ├── accessibility.css # Accessibility helpers
│   ├── layout.css       # Layout utilities
│   └── responsive.css   # Responsive utilities
└── themes/             # Theme-specific styles
    ├── default.css     # Default theme
    ├── dark.css        # Dark theme
    └── high-contrast.css # High contrast theme
```

### Types Directory (`src/types/`)

```
src/types/
├── index.ts              # Main type exports
├── components/           # Component-specific types
│   ├── button.ts        # Button component types
│   ├── form.ts          # Form component types
│   ├── layout.ts        # Layout component types
│   └── animation.ts     # Animation component types
├── theme/               # Theme-related types
│   ├── colors.ts        # Color type definitions
│   ├── spacing.ts       # Spacing type definitions
│   └── typography.ts    # Typography type definitions
├── animation/           # Animation-related types
│   ├── gsap.ts         # GSAP-specific types
│   ├── transitions.ts   # Transition types
│   └── presets.ts      # Animation preset types
└── utility/            # Utility types
    ├── common.ts       # Common utility types
    ├── react.ts        # React-specific utility types
    └── css.ts          # CSS-related types
```

## CLI Directory Structure

```
cli/
├── index.cjs             # Main CLI entry point
├── commands/             # Individual CLI commands
│   ├── init.js          # Project initialization
│   ├── add.js           # Add components
│   ├── update.js        # Update components
│   ├── migrate.js       # Migration utilities
│   └── doctor.js        # Health check diagnostics
├── templates/           # Project templates
│   ├── dashboard/       # Dashboard template
│   ├── landing-page/    # Landing page template
│   └── admin-panel/     # Admin panel template
├── utils/              # CLI utility functions
│   ├── file-system.js  # File system operations
│   ├── package-manager.js # Package manager detection
│   ├── framework-detection.js # Framework detection
│   └── validation.js   # Input validation
└── migrations/         # Version migration scripts
    ├── v2-to-v3.js     # v2 to v3 migration
    └── utils.js        # Migration utilities
```

## Documentation Structure

```
docs/
├── GETTING_STARTED.md   # Quick start guide
├── CONTRIBUTING.md      # Contribution guidelines
├── DEPLOYMENT.md        # Deployment instructions
├── MIGRATION_GUIDES/    # Version migration guides
│   ├── v2-to-v3.md     # v2 to v3 migration
│   └── breaking-changes.md # Breaking changes log
├── API_REFERENCE/       # Component API documentation
│   ├── components/      # Individual component docs
│   ├── hooks/          # Hooks documentation
│   └── utilities/      # Utility function docs
├── DESIGN_SYSTEM/      # Design system documentation
│   ├── colors.md       # Color palette guide
│   ├── typography.md   # Typography system
│   ├── spacing.md      # Spacing system
│   └── animations.md   # Animation guidelines
└── EXAMPLES/           # Usage examples
    ├── basic-usage.md  # Basic usage examples
    ├── advanced.md     # Advanced usage patterns
    └── integrations.md # Framework integrations
```

## Build Output Structure

```
dist/
├── index.js            # CommonJS build
├── index.esm.js        # ES modules build
├── index.d.ts          # TypeScript definitions
├── components/         # Individual component builds
│   ├── Button/         # Button component build
│   ├── Card/           # Card component build
│   └── ...             # Other components
├── styles/             # Compiled CSS styles
│   ├── globals.css     # Global styles
│   ├── components.css  # Component styles
│   └── utilities.css   # Utility styles
└── types/              # Generated type definitions
    ├── components.d.ts # Component types
    ├── hooks.d.ts      # Hook types
    └── utils.d.ts      # Utility types
```

## File Naming Conventions

### Components
- **Component Files**: PascalCase (e.g., `Button.tsx`, `DataTable.tsx`)
- **Component Directories**: PascalCase (e.g., `Button/`, `DataTable/`)
- **Test Files**: `ComponentName.test.tsx`
- **Type Files**: `types.ts` (within component directory)
- **Export Files**: `index.ts` (barrel exports)

### Hooks
- **Hook Files**: camelCase with `use` prefix (e.g., `useZenAnimation.ts`)
- **Hook Directories**: camelCase (e.g., `animation/`, `form/`)

### Utilities
- **Utility Files**: camelCase (e.g., `utils.ts`, `animations.ts`)
- **Utility Directories**: camelCase (e.g., `theme/`, `constants/`)

### Styles
- **CSS Files**: kebab-case (e.g., `globals.css`, `button.css`)
- **Theme Files**: kebab-case (e.g., `dark.css`, `high-contrast.css`)

### Documentation
- **Markdown Files**: SCREAMING_SNAKE_CASE for important docs, kebab-case for others
- **Important Docs**: `README.md`, `CONTRIBUTING.md`, `GETTING_STARTED.md`
- **Regular Docs**: `migration-guide.md`, `api-reference.md`

## Import/Export Patterns

### Barrel Exports
Every directory should have an `index.ts` file for clean imports:

```typescript
// src/components/ui/index.ts
export { Button, type ButtonProps } from './Button';
export { Card, CardHeader, CardContent, type CardProps } from './Card';
export { Input, type InputProps } from './Form/Input';
```

### Component Exports
```typescript
// src/components/ui/Button/index.ts
export { Button, type ButtonProps } from './Button';
export type { ButtonVariant, ButtonSize } from './types';
```

### Hook Exports
```typescript
// src/hooks/index.ts
export { useZenAnimation } from './animation/useZenAnimation';
export { useReducedMotion } from './animation/useReducedMotion';
export { useFormValidation } from './form/useFormValidation';
```

### Utility Exports
```typescript
// src/lib/index.ts
export { cn } from './utils/cn';
export { zenColors } from './theme/colors';
export { ZEN_TIMING, ZEN_EASING } from './animations/timing';
```

## Code Organization Principles

### Single Responsibility
- Each file should have a single, clear purpose
- Components should focus on one specific UI element
- Utilities should handle one specific task

### Dependency Direction
- Components depend on hooks and utilities
- Hooks depend on utilities
- Utilities are dependency-free (except for external libraries)

### Abstraction Levels
- **Low Level**: Utilities, constants, types
- **Mid Level**: Hooks, context providers
- **High Level**: Components, layouts

### Encapsulation
- Keep component internals private
- Expose only necessary interfaces
- Use TypeScript for clear boundaries

This structure ensures consistency, maintainability, and scalability as the project grows.

---
> Source: [ingpoc/zen-flow-ui](https://github.com/ingpoc/zen-flow-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
