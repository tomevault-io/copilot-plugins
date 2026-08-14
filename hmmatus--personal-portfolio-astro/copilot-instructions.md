## project-structure

> ├── components/           # React & Astro components (atomic design)

# Project Structure & Configuration

## Directory Architecture

### Core Structure
```
src/
├── components/           # React & Astro components (atomic design)
│   ├── buttons/         # Atom: Button components
│   ├── cards/           # Molecule: Card components  
│   ├── form/            # Molecule: Form-related components
│   ├── sections/        # Organism: Page sections
│   └── ...
├── layouts/             # Astro layout components
├── pages/               # Astro pages (routing)
├── assets/              # Static assets (icons, images)
├── styles/              # Global styles and theme
├── types/               # TypeScript type definitions
├── utils/               # Utility functions
├── data/                # Static data files
├── consts/              # Application constants
└── i18n/                # Internationalization
```

### Component Organization
Follow atomic design principles:
- **Atoms** (`buttons/`, `form/inputs/`): Basic building blocks
- **Molecules** (`cards/`, `form/`): Simple component groups
- **Organisms** (`sections/`): Complex component compositions

Reference: [ContactButton.tsx](mdc:src/components/buttons/contact/ContactButton.tsx), [ContactSection.tsx](mdc:src/components/sections/contact/ContactSection.tsx)

## Styling Architecture

### Theme System
Centralized theme variables in `src/styles/theme/`:
```scss
// _colors.scss
$primary-background: #000000;
$secondary-background: #1A1A1A;
$primary-text: #FFFFFF;
$secondary: #D3E97A;
$border: #484848;
```

Reference: [_colors.scss](mdc:src/styles/theme/_colors.scss)

### SCSS Module Pattern
Each component has its own style module:
```
ComponentName/
├── ComponentName.tsx
├── ComponentName.module.scss
```

Import theme variables:
```scss
@use "../../../styles/theme/main.scss" as main;

.component {
  background-color: main.$primary-background;
  color: main.$secondary;
}
```

### Responsive Design
Use consistent mixins for breakpoints:
```scss
@import "../../styles/theme/mixins";

.component {
  @include mobile {
    // Mobile styles
  }
}
```

Reference: [_mixins.scss](mdc:src/styles/theme/_mixins.scss)

## TypeScript Configuration

### Type Organization
- **Component Types**: `src/types/` directory
- **Interface Naming**: Use `I` suffix for data interfaces (e.g., `ExperienceI`)
- **Props Interfaces**: Extend HTML element types when appropriate

Example:
```typescript
export interface ExperienceI {
  title: string;
  company: string;
  startDate: string;
  endDate: string;
  description: string;
}
```

Reference: [experience.ts](mdc:src/types/experience.ts)

### Import Patterns
Use path aliases and barrel exports:
```typescript
// Path alias imports
import { Component } from "@components/category";
import { CONSTANT } from "src/consts/constants";

// Barrel exports in index.ts files
export { ContactButton } from "./contact/ContactButton";
export { ContactIconButton } from "./custom/ContactIconButton";
```

## Asset Management

### SVG Icons
Import SVG files as React components:
```typescript
import IconName from "../../../assets/icons/icon-name.svg?react";

// Usage in JSX
<IconName className={styles["icon-class"]} />
```

### Image Organization
Organize images by purpose:
```
public/images/
├── profile/      # Profile/hero images
├── projects/     # Project screenshots
└── ...
```

## Internationalization (i18n)

### Language Structure
Support multiple languages with structured translation files:
```typescript
// src/i18n/ui.ts
export const languages = {
  en: 'English',
  es: 'Español'
};
```

### Component i18n Integration
Handle language switching in components:
```typescript
const handleLanguageChange = (newLang: string) => {
  let path = currentPath;
  if (path.startsWith("/es") || path.startsWith("/en")) {
    path = path.substring(3) || "/";
  }
  const newPath = newLang === "en" ? path : `/${newLang}${path}`;
  window.location.href = newPath;
};
```

Reference: [LanguagePicker.tsx](mdc:src/components/language-picker/LanguagePicker.tsx)

## Constants & Configuration

### Application Constants
Centralize configuration values:
```typescript
// src/consts/social-media-links.ts
export const GITHUB = "https://github.com/username";
export const LINKEDIN = "https://linkedin.com/in/username";

// src/consts/toast-messages.ts
export const TOAST_MESSAGES = {
  SUCCESS: "Message sent successfully!",
  ERROR: "Failed to send message"
};
```

### Theme Constants
For JavaScript/TypeScript usage:
```typescript
export const COLORS = {
  PRIMARY_BACKGROUND: "#000000",
  SECONDARY: "#D3E97A",
  PRIMARY_TEXT: "#FFFFFF"
} as const;
```

## Build & Configuration

### Astro Configuration
Key configuration patterns in `astro.config.mjs`:
- React integration for interactive components
- Path aliases for clean imports
- Build optimizations

### Package Management
Dependencies organization:
- **Core**: Astro, React, TypeScript
- **Styling**: SCSS, CSS modules
- **Utils**: Type-safe utilities
- **Dev Tools**: Linting, formatting

## Performance Considerations

### Component Loading
Use Astro's client directives strategically:
```astro
<!-- Load immediately for critical interactivity -->
<Component client:load />

<!-- Load when visible for performance -->
<Component client:visible />

<!-- Load only on specific breakpoints -->
<Component client:media="(max-width: 768px)" />
```

### Code Splitting
Organize components to enable effective code splitting:
- Lazy load heavy components
- Separate by feature/page
- Use dynamic imports for large dependencies

## Development Workflow

### File Creation Checklist
When creating new components:
1. ✅ Create component directory
2. ✅ Add TypeScript component file
3. ✅ Add SCSS module file
4. ✅ Define proper TypeScript interfaces
5. ✅ Add to barrel exports (index.ts)
6. ✅ Import theme variables in styles
7. ✅ Include accessibility attributes
8. ✅ Test responsive behavior

### Naming Conventions
- **Files**: PascalCase for components (`ContactButton.tsx`)
- **Directories**: kebab-case for feature groups (`contact/`)
- **CSS Classes**: kebab-case (`contact-button`)
- **Variables**: camelCase for JavaScript, kebab-case for CSS

This structure ensures maintainable, scalable, and performant component development within the Astro ecosystem.

---
> Source: [hmmatus/personal-portfolio-astro](https://github.com/hmmatus/personal-portfolio-astro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
