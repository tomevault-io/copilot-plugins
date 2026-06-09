## frontend-contab-2026

> This is an Angular 21 application using standalone components, signals, and server-side rendering (SSR). The project uses Vitest for testing and Prettier for code formatting.

# AGENTS.md - Developer Guide for posinvent

## Overview

This is an Angular 21 application using standalone components, signals, and server-side rendering (SSR). The project uses Vitest for testing and Prettier for code formatting.

## Build Commands

```bash
# Development server (http://localhost:4200)
npm start
# or: ng serve

# Build for production
npm run build
# or: ng build

# Watch mode (development builds on file changes)
npm run watch
# or: ng build --watch --configuration development

# Run unit tests (Vitest)
npm test
# or: ng test

# Run SSR server
npm run serve:ssr:posinvent
```

### Running a Single Test

To run a specific test file, use Vitest directly:

```bash
npx vitest run src/app/app.spec.ts
```

Or filter by test name:

```bash
npx vitest run --testNamePattern="should create the app"
```

### Code Generation

```bash
# Generate a new component
ng generate component component-name

# Generate a service
ng generate service service-name

# Generate other schematics
ng generate --help
```

## Code Style Guidelines

### Formatting

- **Line width**: 100 characters (configured in `.prettierrc`)
- **Quotes**: Single quotes for TypeScript, double quotes for HTML
- **Indentation**: 2 spaces
- **Trailing whitespace**: Trimmed
- **Final newline**: Always present
- **Run Prettier before committing**: `npx prettier --write .`

### TypeScript Configuration

The project uses strict TypeScript with these settings (`tsconfig.json`):

- `strict: true` - full strict type checking
- `noImplicitOverride: true` - requires `override` keyword
- `noPropertyAccessFromIndexSignature: true` - strict property access
- `noImplicitReturns: true` - all code paths must return
- `noFallthroughCasesInSwitch: true` - exhaustive switch cases
- `isolatedModules: true` - safe transpilation
- `experimentalDecorators: true` - Angular decorators
- `target: ES2022` - modern JavaScript

### Angular Patterns

#### Standalone Components

Use Angular 17+ standalone component pattern (no NgModules):

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-example',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './example.html',
  styleUrl: './example.css',
})
export class Example {
  // Use signals for reactive state
  readonly count = signal(0);
}
```

#### Signals

Prefer signals over RxJS for component state:

```typescript
readonly title = signal('initial value');

updateTitle(newValue: string) {
  this.title.set(newValue);
}

// Or computed
readonly derived = computed(() => this.title().toUpperCase());
```

#### Dependency Injection

Inject services via constructor:

```typescript
constructor(private readonly myService: MyService) {}
```

Use `providedIn: 'root'` for application-wide singletons.

### Naming Conventions

- **Components**: `PascalCase` (e.g., `UserProfileComponent`)
- **Services**: `PascalCase` (e.g., `InventoryService`)
- **Files**: `kebab-case` (e.g., `user-profile.component.ts`)
- **Interfaces**: `PascalCase` (e.g., `User`)
- **Constants**: `UPPER_SNAKE_CASE` for enum values, `camelCase` otherwise

### Imports

- Organize imports in this order:
  1. Angular core (e.g., `@angular/core`)
  2. Third-party libraries
  3. Relative imports (grouped by depth)
- Use absolute paths for Angular packages, relative for app code

```typescript
// Good
import { Component, signal, inject } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { map } from 'rxjs/operators';

import { MyService } from './services/my.service';
import { UserCardComponent } from './components/user-card/user-card.component';
```

### Error Handling

- Use try/catch for async operations
- Display user-friendly error messages in UI
- Log errors to console with context
- Consider Angular's `ErrorHandler` for global handling

### Testing

Tests use Vitest with Angular TestBed:

```typescript
import { TestBed } from '@angular/core/testing';
import { MyComponent } from './my.component';

describe('MyComponent', () => {
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [MyComponent],
    }).compileComponents();
  });

  it('should create', () => {
    const fixture = TestBed.createComponent(MyComponent);
    const component = fixture.componentInstance;
    expect(component).toBeTruthy();
  });
});
```

### File Organization

```
src/
├── app/
│   ├── components/      # Reusable UI components
│   ├── services/        # Business logic services
│   ├── models/          # TypeScript interfaces/types
│   ├── guards/          # Route guards
│   └── interceptors/    # HTTP interceptors
├── assets/              # Static assets
└── styles.css          # Global styles
```

### Additional Notes

- SSR is enabled - test server-side rendering behavior
- Node types included for server-side code
- Use `readonly` modifier where possible
- Avoid `any` - prefer proper types or `unknown`

---
> Source: [Jaminson0374/frontend_contab_2026](https://github.com/Jaminson0374/frontend_contab_2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
