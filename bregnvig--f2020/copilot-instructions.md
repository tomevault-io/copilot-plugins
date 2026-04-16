## f2020

> F2020 is a Formula 1 betting application built with Angular and Firebase. The application allows users to place bets on F1 races and tracks live results using OpenF1 API data.

# F2020 Project Context for Gemini

## Project Overview

F2020 is a Formula 1 betting application built with Angular and Firebase. The application allows users to place bets on F1 races and tracks live results using OpenF1 API data.

## Tech Stack

- **Frontend**: Angular 19, Angular Material, TailwindCSS
- **Backend**: Firebase (Firestore, Functions, Cloud Messaging)
- **Build System**: Nx monorepo
- **APIs**: OpenF1 for live F1 data

## Key Commands

### Development

- `npm run start` or `nx serve ui` - Start UI development server
- `nx serve builder` - Run data builder (populate Firestore)
- `npm run serve:firebase` - Start Firebase functions development

### Testing & Quality

- `npm test` - Run tests
- `npm run lint` - Run linting
- `npm run format` - Format code

### Firebase/Emulator

- `firebase emulators:start --only=functions,firestore,auth,pubsub --config=firebase.json --export-on-exit=./saved-data --import=./saved-data --inspect-functions --project f1-playground-e1f23`
- `npm run kill-ports` - Kill occupied ports

### Build & Deploy

- `npm run build:ui` - Build UI for production
- `npm run deploy:all` - Deploy hosting, functions, and rules
- `npm run deploy:hosting` - Deploy hosting only
- `npm run deploy:functions` - Deploy functions only

## Project Structure

- `apps/ui/` - Angular frontend application
- `apps/functions/` - Firebase Cloud Functions
- `apps/builder/` - Data import/migration utilities
- `libs/` - Shared libraries and feature modules
  - `libs/api/` - API services and data layer
  - `libs/data/` - Data models and mappers
  - `libs/openf1/` - OpenF1 API types
  - `libs/shared/` - Shared UI components
  - Other feature-specific libraries (race, player, standing, etc.)

## Key Services

- `LiveResultService` - Handles live race data and WebSocket connections
- `OpenF1HttpService` - HTTP client for OpenF1 API
- `OpenF1WSSService` - WebSocket service for real-time F1 data

## Development Notes

- Uses Nx for monorepo management
- Firebase emulator for local development
- OpenF1 API integration for live race data
- WebSocket connections for real-time updates
- Angular standalone components pattern

## TypeScript & Functional Programming Style

This project emphasizes a modern, functional, and idiomatic TypeScript style. We aim to write code that is not only correct but also clean, expressive, and leverages the full power of the language. Avoid writing TypeScript as if it were a class-based language like Java or C#.

### Embrace Functional Constructs

- **Use `map`, `reduce`, `filter`**: Prefer functional array methods over imperative loops (`for`, `while`). This leads to more declarative and readable code.

  ```typescript
  // ✅ PREFERRED: Functional and declarative
  const numbers = [1, 2, 3, 4];
  const doubledAndSummed = numbers
    .map(n => n * 2)
    .reduce((sum, n) => sum + n, 0);

  // ❌ AVOID: Imperative loop
  let sum = 0;
  for (const n of numbers) {
    sum += n * 2;
  }
  ```

- **Immutability**: Treat data as immutable where possible. Instead of modifying objects and arrays directly, create new ones with the updated values.

  ```typescript
  // ✅ PREFERRED: Immutable update
  const user = { id: 1, name: 'John' };
  const updatedUser = { ...user, name: 'Jane' };

  // ❌ AVOID: Direct mutation
  const userToUpdate = { id: 1, name: 'John' };
  userToUpdate.name = 'Jane'; // Mutation
  ```

### Leverage Modern ESNext Features

- **Optional Chaining (`?.`) and Nullish Coalescing (`??`)**: Use these operators to handle potentially null or undefined values gracefully and concisely.

  ```typescript
  // ✅ PREFERRED: Clean and safe
  const userName = user?.profile?.name ?? 'Guest';

  // ❌ AVOID: Verbose and error-prone checks
  let userNameResult = 'Guest';
  if (user && user.profile && user.profile.name) {
    userNameResult = user.profile.name;
  }
  ```

### Write "Real" TypeScript

- **Strong Typing**: Leverage TypeScript's type system. Avoid `any` whenever possible. Define clear interfaces and types for your data structures.
- **Code with a TypeScript Mindset**: Don't just write JavaScript and add types later. Think in terms of types from the beginning. This helps in designing better APIs and components.
- **Keep it Lean**: Avoid overly complex class hierarchies or design patterns borrowed from languages like Java/C# if a simpler functional approach will suffice. TypeScript is a multi-paradigm language; use the right tool for the job.

## Angular Coding Standards

### Functional API (Preferred)

**Always use Angular's functional API instead of decorators**:

```typescript
import { booleanAttribute } from '@angular/core';

// ✅ PREFERRED: Functional API
export class MyComponent {
  placeholder = input<string>();
  disabled = input<boolean, string | boolean>(false, { transform: booleanAttribute });
  valueChange = output<string>();
}

// ❌ AVOID: Decorator API
export class MyComponentWithDecorators {
  @Input() placeholder?: string;
  @Input() disabled = false;
  @Output() valueChange = new EventEmitter<string>();
}
```

**Benefits of Functional API**:

- Better type inference
- Easier to test and mock
- More consistent with modern Angular patterns
- Better tree-shaking

**IMPORTANT: Boolean Input Requirements**:

- **ALWAYS** use `booleanAttribute` transformer for boolean inputs
- **NEVER** leave constructors empty - always include initialization logic
- **When refactoring**: If you find an empty constructor, remove it entirely
- This ensures proper HTML attribute handling (e.g., `<component disabled>` works correctly)

```typescript
import { booleanAttribute, effect, input } from '@angular/core';

// ✅ CORRECT: Boolean input with transformer
export class MyComponent {
  disabled = input<boolean, string | boolean>(false, { transform: booleanAttribute });

  constructor() {
    // Required: Never leave constructor empty
    effect(() => {
      // Initialization logic here
    });
  }
}

// ❌ WRONG: Boolean input without transformer
export class MyComponentWrong {
  disabled = input<boolean>(false); // Missing booleanAttribute

  constructor() {
    // Empty constructor - NOT ALLOWED
  }
}

// ✅ CORRECT: When refactoring, remove empty constructor entirely
export class RefactoredComponent {
  disabled = input<boolean, string | boolean>(false, { transform: booleanAttribute });

  // No constructor needed if no initialization logic
}
```

### Input Requirements and User Prompting

**When creating components with inputs, follow these guidelines for required vs optional properties**:

```typescript
import { booleanAttribute, effect, input } from '@angular/core';

// ✅ PREFERRED: Clear distinction between required and optional inputs
export class MyComponentWithInputs {
  // Required inputs - no default value
  userId = input.required<string>();
  entityId = input.required<number>();

  // Optional inputs - provide sensible defaults
  placeholder = input<string>('Enter value...');
  disabled = input<boolean, string | boolean>(false, { transform: booleanAttribute });
  maxLength = input<number>(100);

  constructor() {
    // Never leave constructor empty - always include initialization logic
    effect(() => {
      console.log('Component initialized with userId:', this.userId());
    });
  }
}
```

**When to ask users for clarification**:

- **ASK** when a required input has no clear default and the context doesn't provide obvious guidance
- **ASK** when the business logic requires specific validation rules or constraints
- **ASK** when there are multiple valid approaches and the choice affects functionality significantly

**When to make reasonable assumptions**:

- **ASSUME** sensible defaults for optional inputs (empty strings, false for booleans, reasonable numbers)
- **ASSUME** standard Angular patterns (e.g., disabled defaults to false, placeholder defaults to helpful text)
- **ASSUME** consistent behavior with existing similar components in the codebase

**Examples**:

```typescript
import { input } from '@angular/core';
import { ValidatorFn } from '@angular/forms';

// When in doubt about validation - ASK USER
export class FormInputComponent {
  value = input.required<string>();
  // Should ask: "What validation rules do you need? (required, email, minLength, etc.)"
  validators = input<ValidatorFn[]>([]);
}

// Standard UI patterns - MAKE ASSUMPTIONS
export class ButtonComponent {
  label = input.required<string>();
  disabled = input<boolean, string | boolean>(false, { transform: booleanAttribute });
  type = input<'button' | 'submit'>('button'); // Sensible default
  variant = input<'primary' | 'secondary'>('primary'); // Reasonable default

  constructor() {
    // Always include constructor logic - never leave empty
    effect(() => {
      if (this.disabled()) {
        console.log('Button is disabled');
      }
    });
  }
}
```

### New Control Flow Syntax (Preferred)

**Always use the new `@if`, `@for`, `@switch` syntax**:

```typescript
import { Component } from '@angular/core';

// ✅ PREFERRED: New control flow
@Component({
  standalone: true,
  template: `
    @if (isLoading) {
      <div>Loading...</div>
    } @else {
      <div>Content loaded</div>
    }

    @for (item of items; track item.id) {
      <div>{{ item.name }}</div>
    } @empty {
      <div>No items found</div>
    }

    @switch (status) {
      @case ('pending') {
        <div>Pending...</div>
      }
      @case ('complete') {
        <div>Complete!</div>
      }
      @default {
        <div>Unknown status</div>
      }
    }
  `
})
export class NewControlFlowComponent {
  isLoading = false;
  items: {id: number, name: string}[] = [];
  status = 'pending';
}

// ❌ AVOID: Old structural directives
@Component({
  standalone: true,
  template: `
    <div *ngIf="isLoading; else content">Loading...</div>
    <ng-template #content>Content loaded</ng-template>

    <div *ngFor="let item of items; trackBy: trackByFn">
      {{ item.name }}
    </div>

    <div [ngSwitch]="status">
      <div *ngSwitchCase="'pending'">Pending...</div>
      <div *ngSwitchCase="'complete'">Complete!</div>
      <div *ngSwitchDefault>Unknown status</div>
    </div>
  `
})
export class OldControlFlowComponent {
  isLoading = false;
  items: {id: number, name: string}[] = [];
  status = 'pending';
  trackByFn(index: number, item: {id: number, name: string}) {
    return item.id;
  }
}
```

**Benefits of New Control Flow**:

- Better type checking
- Improved performance
- More readable and maintainable
- Better IDE support

### Private Fields

**Use `#` modifier for true private fields instead of `private` keyword**:

```typescript
import { inject, signal } from '@angular/core';

// ✅ PREFERRED: True private fields with #
export class MyComponentWithPrivateFields {
  #service = inject(MyService);
  #internalState = signal('initial');

  constructor() {
    // Can access #service and #internalState here
  }

  publicMethod() {
    return this.#internalState();
  }
}

// ❌ AVOID: TypeScript private keyword
export class MyComponentWithTSPrivate {
  private service = inject(MyService);
  private internalState = signal('initial');

  constructor() {
    // These are not truly private at runtime
  }
}
```

**Benefits of `#` Private Fields**:

- True privacy at runtime (not just compile-time)
- Cannot be accessed from outside the class, even with type assertions
- Better encapsulation and security
- Follows modern JavaScript standards

### Dependency Injection

**Use `inject()` function instead of constructor injection**:

```typescript
import { inject } from '@angular/core';
import { FormBuilder, Validators } from '@angular/forms';
import { Router } from '@angular/router';

// ✅ PREFERRED: inject() function with private fields at top
export class MyComponentWithInject {
  #service = inject(MyService);
  #router = inject(Router);
  #fb = inject(FormBuilder);

  // Other class members after injected dependencies
  form = this.#fb.group({
    name: ['', Validators.required],
  });

  constructor() {
    // Constructor logic if needed
  }

  someMethod() {
    this.#service.getData();
  }
}

// ❌ AVOID: Constructor injection
export class MyComponentWithConstructorDI {
  constructor(private service: MyService, private router: Router, private fb: FormBuilder) {}
}
```

**Benefits of `inject()` Function**:

- More concise and readable
- Better tree-shaking
- Easier testing and mocking
- Works well with functional programming patterns
- Can be used outside of constructor

**Ordering Rules**:

- Static fields/functions first (if any)
- Injected dependencies with `#` at the very top of class body
- Other class members after injected dependencies

### Signal-Based Reactive Programming

**Use signals for state management**:

```typescript
import { computed, signal } from '@angular/core';

// ✅ PREFERRED: Signals
export class MyComponentWithSignals {
  count = signal(0);
  doubledCount = computed(() => this.count() * 2);

  increment() {
    this.count.update((c) => c + 1);
  }
}

// ❌ AVOID: Traditional reactive forms without signals
export class MyComponentWithoutSignals {
  count = 0;

  get doubledCount() {
    return this.count * 2;
  }
}

// Dummy classes for compilation
class MyService { getData() {} }
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bregnvig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
