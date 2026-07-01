## angularvibecoding

> This is a modern Angular 20 application using standalone components, signals, TailwindCSS, and DaisyUI. The project is optimized for AI-assisted development and follows strict best practices.

# Angular 20 VibeCoding Project - Cursor Rules

## Project Overview

This is a modern Angular 20 application using standalone components, signals, TailwindCSS, and DaisyUI. The project is optimized for AI-assisted development and follows strict best practices.

## Core Technologies

- **Angular 20+** with standalone architecture (no NgModules)
- **TypeScript** with strict type checking
- **Angular Signals** for reactive state management
- **TailwindCSS 4** + **DaisyUI** for styling
- **SCSS** as the primary styling engine
- **RxJS** for complex asynchronous operations only
- **Angular Material** for additional UI components

## Critical Naming Conventions

⚠️ **IMPORTANT**: This project does NOT use standard Angular file suffixes

- ✅ Use: `login.ts`, `auth.ts`, `notification.ts`
- ❌ Never use: `login.component.ts`, `auth.service.ts`, `notification.service.ts`
- Components, services, guards, and pipes all use simple `.ts` extension
- Each feature has its own folder with `.ts`, `.html`, `.scss`, and `.spec.ts` files

## File Structure Pattern

```
feature-name/
  ├── feature-name.ts        # Component or service
  ├── feature-name.html      # Template (if component)
  ├── feature-name.scss      # Styles (if component)
  └── feature-name.spec.ts   # Tests
```

## TypeScript Best Practices

### Always Use Strict Typing

- ❌ NEVER use `any` type
- ✅ Define proper interfaces for all data structures
- ✅ Use type inference where obvious
- ✅ Use `unknown` for uncertain types (rare cases)

```typescript
// ✅ Good
interface LoginCredentials {
  email: string;
  password: string;
}

// ❌ Bad
const credentials: any = { email: "", password: "" };
```

## Angular Component Best Practices

### Component Structure

```typescript
import {
  Component,
  signal,
  computed,
  inject,
  input,
  output,
} from "@angular/core";
import { ChangeDetectionStrategy } from "@angular/core";

@Component({
  selector: "app-example",
  templateUrl: "./example.html",
  styleUrls: ["./example.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    /* standalone imports */
  ],
})
export class Example {
  // Use inject() instead of constructor injection
  private router = inject(Router);
  private authService = inject(Auth);

  // Use input() and output() functions instead of decorators
  title = input<string>("Default Title");
  users = input<User[]>([]);
  formSubmitted = output<FormData>();

  // Use signals for local state
  isLoading = signal(false);
  isActive = signal(false);

  // Use computed() for derived state
  isFormValid = computed(() => !this.nameError() && !this.emailError());
  backgroundColor = computed(() => (this.isActive() ? "#e3f2fd" : "#ffffff"));
}
```

### Key Component Rules

1. ✅ All components use `ChangeDetectionStrategy.OnPush`
2. ✅ Use `inject()` function instead of constructor injection
3. ✅ Use `input()` and `output()` functions instead of `@Input()` and `@Output()` decorators
4. ✅ Use signals for all local state
5. ✅ Use `computed()` for all derived state
6. ✅ Keep components small and focused (single responsibility)
7. ✅ All components are standalone (no NgModules)

## Template Best Practices

### Use Native Control Flow (NOT *ngIf, *ngFor, \*ngSwitch)

```html
<!-- ✅ Good: Use native control flow -->
@if (isVisible()) {
<h2>{{ title() }}</h2>
} @for (user of users(); track user.id) {
<div class="user-item">
  <span>{{ user.name }}</span>
</div>
} @switch (status()) { @case ('loading') {
<div class="loading">Loading...</div>
} @case ('success') {
<div class="success">Success!</div>
} @default {
<div class="default">Default state</div>
} }

<!-- ❌ Bad: Don't use old syntax -->
<div *ngIf="isVisible()">...</div>
<div *ngFor="let user of users()">...</div>
```

### Use Class and Style Bindings (NOT ngClass/ngStyle)

```html
<!-- ✅ Good: Use direct bindings -->
<div [class.active]="isActive()" [class.disabled]="isDisabled()">
  Dynamic classes
</div>

<div [style.background-color]="backgroundColor()" [style.color]="textColor()">
  Dynamic styles
</div>

<!-- ❌ Bad: Don't use ngClass/ngStyle -->
<div [ngClass]="{'active': isActive()}">...</div>
<div [ngStyle]="{'background-color': backgroundColor()}">...</div>
```

### Always Use Async Pipe for Observables

```html
<!-- ✅ Good -->
@if (user$ | async; as user) {
<div>{{ user.name }}</div>
}

<!-- ❌ Bad: Don't manually subscribe in templates -->
```

## Service Best Practices

### Service Structure

```typescript
import { Injectable, inject, signal } from "@angular/core";

@Injectable({
  providedIn: "root", // Always use root provider
})
export class Auth {
  // Use inject() instead of constructor injection
  private router = inject(Router);
  private notification = inject(NotificationService);

  // Use signals for service state
  currentUser = signal<User | null>(null);
  isAuthenticated = computed(() => this.currentUser() !== null);

  // Service methods...
}
```

### Key Service Rules

1. ✅ All services use `providedIn: 'root'`
2. ✅ Use `inject()` function instead of constructor injection
3. ✅ Single responsibility principle
4. ✅ Use signals for state management

## State Management

### Use Signals for State

```typescript
// ✅ Local component state
isSubmitting = signal(false);
errorMessage = signal("");

// ✅ Computed state
isFormValid = computed(
  () => this.email().length > 0 && this.password().length > 0
);

// ✅ Effects for side effects
effect(() => {
  if (this.isAuthenticated()) {
    this.router.navigate(["/dashboard"]);
  }
});
```

### State Rules

1. ✅ Use signals for all local state
2. ✅ Use computed() for all derived state
3. ✅ Use effect() for side effects
4. ✅ Keep state transformations pure and predictable
5. ✅ Use RxJS only for complex asynchronous operations

## Styling Best Practices

### Use TailwindCSS + DaisyUI in Templates

```html
<!-- ✅ Use Tailwind and DaisyUI classes directly -->
<div class="hero min-h-screen bg-base-200">
  <button class="btn btn-primary">Click me</button>
</div>
```

### SCSS for Component-Specific Styles

```scss
// Use SCSS for complex component-specific styling
.custom-component {
  @apply flex items-center gap-4;

  &__title {
    @apply text-2xl font-bold;
  }
}
```

## Forms Best Practices

### Use Reactive Forms with Signals

```typescript
import { FormControl, FormGroup, Validators } from "@angular/forms";

// ✅ Reactive forms
loginForm = new FormGroup({
  email: new FormControl("", [Validators.required, Validators.email]),
  password: new FormControl("", [Validators.required, Validators.minLength(6)]),
});

// Use signals for form state
isSubmitting = signal(false);
```

## Project Structure

```
src/app/
├── auth/                    # Authentication feature
│   ├── guards/             # Route guards (auth-guard.ts)
│   ├── services/           # Auth services (auth.ts)
│   ├── login/              # Login component
│   ├── register/           # Register component
│   └── user-profile/       # User profile component
├── core/                   # Core application components
│   ├── layout/            # Main layout
│   ├── header/            # Application header
│   ├── footer/            # Application footer
│   ├── sidenav/           # Navigation sidebar
│   └── home/              # Home page
└── shared/                # Shared components and services
    ├── services/          # Shared services
    ├── notification/      # Notification components
    └── spinner/           # Loading spinner
```

## Migration Checklist for New Components

When creating new components or services, ensure:

- [ ] Use `inject()` instead of constructor injection
- [ ] Set `changeDetection: ChangeDetectionStrategy.OnPush`
- [ ] Use signals for local state
- [ ] Use `computed()` for derived state
- [ ] Use `input()` and `output()` functions
- [ ] Use native control flow (`@if`, `@for`, `@switch`)
- [ ] Use class and style bindings instead of `ngClass`/`ngStyle`
- [ ] Define proper TypeScript interfaces
- [ ] Avoid `any` types
- [ ] Use reactive forms with signals
- [ ] Keep components small and focused
- [ ] Use `providedIn: 'root'` for services
- [ ] No `.component.ts` or `.service.ts` suffixes

## Performance Optimizations

- ✅ OnPush change detection on all components
- ✅ Signal-based reactivity
- ✅ Lazy loading for feature routes
- ✅ Track functions in @for loops
- ✅ @defer for lazy loading templates

## Error Handling

- ✅ Proper error handling in async operations
- ✅ User-friendly error messages via notification service
- ✅ Type-safe error interfaces

## Code Generation Commands

```bash
# Generate new component (remember: no .component.ts suffix!)
ng generate component path/component-name

# Generate new service (remember: no .service.ts suffix!)
ng generate service path/service-name

# Generate new guard
ng generate guard path/guard-name

# Clean the template (remove demo/example content)
npm run clean-template
```

## Clean Template Script - Two Modes

This project includes scripts to remove demo content with TWO modes:

### Dashboard Mode (Recommended)

```bash
npm run clean-template:dashboard
```

**Perfect for:** Dashboards, admin panels, full-featured apps

**Keeps:**

- Complete layout structure (header, footer, sidenav, layout)
- Dashboard-style welcome page
- Collapsible navigation
- Professional structure

### Blank Mode (Minimal)

```bash
npm run clean-template:blank
```

**Perfect for:** Custom designs, landing pages, absolute scratch

**Removes:**

- ALL layout components (header, footer, sidenav, layout)
- Gives minimal single-page starter

### Interactive Selection

```bash
npm run clean-template
```

Prompts you to choose between modes.

**Both modes remove:**

- Authentication pages and services
- Demo data pages
- Example components

**Both modes keep:**

- Shared utilities (notification, spinner, services)
- All configuration files
- TailwindCSS and DaisyUI setup
- Documentation and .cursorrules
- Best practices structure

📖 See [docs/clean-template-modes.md](docs/clean-template-modes.md) for detailed comparison.

## Additional Context

- This project is designed for AI-assisted development
- Code should be clean, scalable, and maintainable
- Follow the example component at `src/app/shared/example-component/example-component.ts`
- Refer to `docs/best-practices.md` for detailed examples
- Use `docs/setup.md` for AI context setup guidelines

## AI/LLM Integration Patterns

When building AI-powered features with Angular, follow these patterns from [Angular.dev AI Design Patterns](https://angular.dev/ai/design-patterns):

### Triggering AI Requests with Signals

**Pattern: Separate input from submission**

```typescript
// Store user's raw input
userInput = signal('');

// Store submitted value that triggers API
submittedPrompt = signal('');

// Resource triggered only on submission
aiResource = resource({
  params: () => this.submittedPrompt(),
  loader: async ({params}) => {
    return await this.aiService.generateContent(params);
  }
});

// On submit button click
onSubmit() {
  this.submittedPrompt.set(this.userInput());
}
```

### Managing LLM Response Data

**Pattern: Use linkedSignal for accumulating responses**

```typescript
// Accumulate chat history or streaming responses
chatHistory = linkedSignal<Message[], Message[]>({
  source: () => this.aiResource.value().messages,
  computation: (newMessages, previous) => {
    const existing = previous?.value || [];
    return [...existing, ...newMessages];
  },
});
```

### Streaming AI Responses

**Pattern: Use resource stream for real-time updates**

```typescript
streamingResponse = resource({
  stream: async () => {
    const data = signal<ResourceStreamItem<string>>({ value: "" });
    const response = await this.aiService.streamContent(prompt);

    (async () => {
      for await (const chunk of response.stream) {
        data.update((prev) => {
          if ("value" in prev) {
            return { value: `${prev.value}${chunk}` };
          }
          return prev;
        });
      }
    })();

    return data;
  },
});
```

### AI-Friendly Templates

```html
<!-- Loading state -->
@if (aiResource.isLoading()) {
<div class="loading">
  <span class="loading loading-spinner"></span>
  <p>AI is thinking...</p>
</div>
}

<!-- Success state with streaming content -->
@else if (aiResource.hasValue()) {
<div class="ai-response">{{ aiResource.value() }}</div>
}

<!-- Error state with retry -->
@else if (aiResource.error()) {
<div class="alert alert-error">
  <p>{{ aiResource.error() }}</p>
  <button class="btn btn-primary" (click)="aiResource.reload()">Retry</button>
</div>
}
```

### Performance Best Practices for AI Features

1. **Scoped Resources**: Place AI resources in components that directly use the data
2. **SSR with Hydration**: Show placeholders, defer AI content until hydration
3. **Loading Indicators**: Always show loading state for LLM operations
4. **Error Handling**: Provide retry mechanisms (use `resource.reload()`)
5. **Zoneless Mode**: Use signals to avoid unnecessary change detection

### Type Safety for AI Responses

```typescript
// Define strong types for LLM responses
interface AIStoryResponse {
  storyParts: string[];
  metadata: {
    tokensUsed: number;
    model: string;
  };
}

// Typed resource
storyResource = resource<AIStoryResponse>({
  params: () => this.storyPrompt(),
  loader: async ({ params }) => {
    return await this.aiService.generateStory(params);
  },
});
```

### AI Service Pattern

```typescript
@Injectable({
  providedIn: "root",
})
export class AIService {
  private http = inject(HttpClient);

  // Session management with signals
  sessionId = signal<string | null>(null);

  generateContent(prompt: string) {
    return this.http.post<AIResponse>("/api/ai/generate", {
      prompt,
      sessionId: this.sessionId(),
    });
  }

  async *streamContent(prompt: string) {
    // Streaming implementation
    const response = await fetch("/api/ai/stream", {
      method: "POST",
      body: JSON.stringify({ prompt }),
    });

    const reader = response.body?.getReader();
    if (!reader) return;

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      yield new TextDecoder().decode(value);
    }
  }
}
```

## Remember

- 🚫 NO `.component.ts` or `.service.ts` suffixes
- 🚫 NO `*ngIf`, `*ngFor`, or `*ngSwitch`
- 🚫 NO `ngClass` or `ngStyle`
- 🚫 NO `any` types
- 🚫 NO constructor injection
- 🚫 NO `@Input()` or `@Output()` decorators
- ✅ USE signals, computed, and effect
- ✅ USE inject() function
- ✅ USE input() and output() functions
- ✅ USE native control flow (@if, @for, @switch)
- ✅ USE OnPush change detection
- ✅ USE strict TypeScript
- ✅ USE resource API for AI/async operations
- ✅ USE linkedSignal for accumulating AI responses
- ✅ USE proper loading/error states for AI features

---
> Source: [VibeNights/angularvibecoding](https://github.com/VibeNights/angularvibecoding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
