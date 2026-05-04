## electrobun-rms

> This project uses **Ultracite**, enforcing strict code quality standards through automated formatting and linting. Additionally, this guide covers **OAT UI components**, **React performance best practices**, and **folder structure conventions** for consistent, high-quality code.


# Ultracite Code Standards & LLM Development Guide

This project uses **Ultracite**, enforcing strict code quality standards through automated formatting and linting. Additionally, this guide covers **OAT UI components**, **React performance best practices**, and **folder structure conventions** for consistent, high-quality code.

---

## Quick Reference

- **Format code**: `bun x ultracite fix`
- **Check for issues**: `bun x ultracite check`
- **Diagnose setup**: `bun x ultracite doctor`

Biome (the underlying engine) provides robust linting and formatting. Most issues are automatically fixable.

---

## Part 1: Ultracite Code Standards

### Core Principles

Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

#### Type Safety & Explicitness

- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

#### Modern JavaScript/TypeScript

- Use arrow functions for callbacks and short functions
- Prefer `for...of` loops over `.forEach()` and indexed `for` loops
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safer property access
- Prefer template literals over string concatenation
- Use destructuring for object and array assignments
- Use `const` by default, `let` only when reassignment is needed, never `var`

#### Async & Promises

- Always `await` promises in async functions - don't forget to use the return value
- Use `async/await` syntax instead of promise chains for better readability
- Handle errors appropriately in async code with try-catch blocks
- Don't use async functions as Promise executors

#### React & JSX

- Use function components over class components
- Call hooks at the top level only, never conditionally
- Specify all dependencies in hook dependency arrays correctly
- Use the `key` prop for elements in iterables (prefer unique IDs over array indices)
- Nest children between opening and closing tags instead of passing as props
- Don't define components inside other components
- Use semantic HTML and ARIA attributes for accessibility:
  - Provide meaningful alt text for images
  - Use proper heading hierarchy
  - Add labels for form inputs
  - Include keyboard event handlers alongside mouse events
  - Use semantic elements (`<button>`, `<nav>`, etc.) instead of divs with roles

#### Error Handling & Debugging

- Remove `console.log`, `debugger`, and `alert` statements from production code
- Throw `Error` objects with descriptive messages, not strings or other values
- Use `try-catch` blocks meaningfully - don't catch errors just to rethrow them
- Prefer early returns over nested conditionals for error cases

#### Code Organization

- Keep functions focused and under reasonable cognitive complexity limits
- Extract complex conditions into well-named boolean variables
- Use early returns to reduce nesting
- Prefer simple conditionals over nested ternary operators
- Group related code together and separate concerns

#### Security

- Add `rel="noopener"` when using `target="_blank"` on links
- Avoid `dangerouslySetInnerHTML` unless absolutely necessary
- Don't use `eval()` or assign directly to `document.cookie`
- Validate and sanitize user input

#### Performance

- Avoid spread syntax in accumulators within loops
- Use top-level regex literals instead of creating them in loops
- Prefer specific imports over namespace imports
- Avoid barrel files (index files that re-export everything)
- Use proper image components (e.g., Next.js `<Image>`) over `<img>` tags

---

## Part 2: React Performance Best Practices

### Priority 1: Eliminating Waterfalls (CRITICAL)

**Goal**: Prevent sequential async operations that could run in parallel.

- **async-parallel**: Use `Promise.all()` for independent async operations
  ```typescript
  // ✗ Bad: Sequential
  const user = await fetchUser();
  const posts = await fetchPosts();
  
  // ✓ Good: Parallel
  const [user, posts] = await Promise.all([
    fetchUser(),
    fetchPosts(),
  ]);
  ```

- **async-defer-await**: Move `await` into branches where actually used
  ```typescript
  // ✗ Bad: Awaiting upfront
  const data = await fetchData();
  if (condition) {
    useData(data);
  }
  
  // ✓ Good: Defer await to usage
  const dataPromise = fetchData();
  if (condition) {
    const data = await dataPromise;
    useData(data);
  }
  ```

- **async-suspense-boundaries**: Use Suspense to stream content progressively
  ```typescript
  // ✓ Good: Lazy load with Suspense
  const LazyComponent = lazy(() => import('./Heavy'));
  export default function App() {
    return (
      <Suspense fallback={<Loading />}>
        <LazyComponent />
      </Suspense>
    );
  }
  ```

### Priority 2: Bundle Size Optimization (CRITICAL)

- **bundle-barrel-imports**: Import directly from modules, avoid barrel files
  ```typescript
  // ✗ Bad: Barrel imports
  import { Button, Card } from '@/components';
  
  // ✓ Good: Direct imports
  import { Button } from '@/components/button';
  import { Card } from '@/components/card';
  ```

- **bundle-dynamic-imports**: Use `next/dynamic` for large components
  ```typescript
  import dynamic from 'next/dynamic';
  const HeavyComponent = dynamic(() => import('./Heavy'), {
    loading: () => <Skeleton />,
  });
  ```

- **bundle-defer-third-party**: Load analytics/logging after hydration
  ```typescript
  useEffect(() => {
    // Analytics loaded AFTER render
    import('analytics').then(m => m.init());
  }, []);
  ```

### Priority 3: Server-Side Performance (HIGH)

- **server-cache-react**: Use `React.cache()` for per-request deduplication
  ```typescript
  import { cache } from 'react';
  
  const getCachedUser = cache(async (id: string) => {
    return fetch(`/api/users/${id}`);
  });
  ```

- **server-parallel-fetching**: Restructure components to parallelize fetches
  - Fetch at parent level, pass down data
  - Avoid waterfalls where child components fetch their own data

### Priority 4: Re-render Optimization (MEDIUM)

- **rerender-memo**: Extract expensive work into memoized components
  ```typescript
  const ExpensiveList = memo(({ items }: { items: Item[] }) => {
    return items.map(item => <Item key={item.id} {...item} />);
  });
  ```

- **rerender-dependencies**: Use primitive dependencies in effects
  ```typescript
  // ✗ Bad: Object dependency changes every render
  useEffect(() => {
    // ...
  }, [config]); // New object each time
  
  // ✓ Good: Primitive dependency
  useEffect(() => {
    // ...
  }, [config.id]); // Only changes when ID changes
  ```

- **rerender-lazy-state-init**: Pass function to `useState` for expensive values
  ```typescript
  // ✓ Good: Function only called once
  const [state, setState] = useState(() => expensiveInitialization());
  ```

### Priority 5: Rendering Performance (MEDIUM)

- **rendering-conditional-render**: Use ternary, not `&&` for conditionals
  ```typescript
  // ✗ Bad: && can render falsy values
  {isLoading && <Loading />}
  
  // ✓ Good: Ternary is explicit
  {isLoading ? <Loading /> : <Content />}
  ```

- **rendering-hoist-jsx**: Extract static JSX outside components
  ```typescript
  // ✓ Good: Static content outside
  const STATIC_HEADER = <h1>Welcome</h1>;
  
  export default function Page() {
    return (
      <div>
        {STATIC_HEADER}
        <DynamicContent />
      </div>
    );
  }
  ```

- **rendering-content-visibility**: Use CSS for long lists
  ```css
  .list-item {
    content-visibility: auto;
  }
  ```

### Priority 6: JavaScript Performance (LOW-MEDIUM)

- **js-early-exit**: Return early from functions
  ```typescript
  // ✗ Bad: Nested conditions
  function validate(user) {
    if (user) {
      if (user.email) {
        if (isValidEmail(user.email)) {
          return true;
        }
      }
    }
    return false;
  }
  
  // ✓ Good: Early returns
  function validate(user) {
    if (!user?.email) return false;
    if (!isValidEmail(user.email)) return false;
    return true;
  }
  ```

- **js-set-map-lookups**: Use `Set`/`Map` for O(1) lookups
  ```typescript
  // ✗ Bad: O(n) array lookup
  const hasItem = items.includes(id);
  
  // ✓ Good: O(1) Set lookup
  const itemSet = new Set(items);
  const hasItem = itemSet.has(id);
  ```

- **js-cache-function-results**: Cache expensive function results
  ```typescript
  const cache = new Map();
  function expensive(id: string) {
    if (cache.has(id)) return cache.get(id);
    const result = doExpensiveWork(id);
    cache.set(id, result);
    return result;
  }
  ```

---

## Part 3: OAT UI Components

OAT is an ultra-lightweight semantic HTML + CSS component library with **zero dependencies**. Components use native HTML elements and are styled contextually without requiring classes.

### Installation & Setup

Include OAT CSS and JS bundles. No build step required.

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/oat@latest/dist/oat.min.css" />
<script src="https://cdn.jsdelivr.net/npm/oat@latest/dist/oat.min.js"></script>
```

### Core Components Guide

#### Typography (Auto-styled)

No classes needed. Base text elements styled automatically.

```jsx
<h1>Heading 1</h1>
<h2>Heading 2</h2>
<p>
  Paragraph with <strong>bold</strong>, <em>italic</em>, and{' '}
  <a href="#">links</a>
</p>
<code>inline code</code>
<pre>
  <code>block code</code>
</pre>
<blockquote>Blockquote styled automatically.</blockquote>
```

#### Buttons

Use `data-variant` for semantic intent. Use classes for visual styles.

```jsx
// Variants
<button>Primary (default)</button>
<button data-variant="secondary">Secondary</button>
<button data-variant="danger">Danger</button>

// Styles
<button className="outline">Outline</button>
<button className="ghost">Ghost</button>
<button className="small">Small</button>
<button className="large">Large</button>
<button disabled>Disabled</button>

// Button Groups
<menu className="buttons">
  <button className="outline">Left</button>
  <button className="outline">Center</button>
  <button className="outline">Right</button>
</menu>
```

#### Forms

Wrap inputs in `<label>` for proper accessibility. Use `data-field` for layout.

```jsx
<form>
  <label data-field>
    Name
    <input type="text" placeholder="Enter your name" />
  </label>

  <label data-field>
    Email
    <input type="email" placeholder="you@example.com" />
  </label>

  <label data-field>
    Message
    <textarea placeholder="Your message..."></textarea>
  </label>

  <label data-field>
    <input type="checkbox" /> I agree to terms
  </label>

  <fieldset className="hstack">
    <legend>Preference</legend>
    <label>
      <input type="radio" name="pref" /> Option A
    </label>
    <label>
      <input type="radio" name="pref" /> Option B
    </label>
  </fieldset>

  <button type="submit">Submit</button>
</form>
```

#### Input Groups

Combine inputs with buttons using `className="group"`.

```jsx
<fieldset className="group">
  <legend>https://</legend>
  <input type="url" placeholder="subdomain" />
  <select aria-label="Domain">
    <option>.example.com</option>
    <option>.example.net</option>
  </select>
  <button>Go</button>
</fieldset>
```

#### Accordion

Use native `<details>` and `<summary>` for collapsible content.

```jsx
<details>
  <summary>What is this?</summary>
  <p>Details content here.</p>
</details>

<details name="group">
  <summary>Grouped Item 1</summary>
  <p>Only one open at a time with same name.</p>
</details>

<details name="group">
  <summary>Grouped Item 2</summary>
  <p>Click to open, closes the other.</p>
</details>
```

#### Alerts

Use `role="alert"` with `data-variant` for different styles.

```jsx
<div role="alert">
  <strong>Default Alert</strong> Standard message.
</div>

<div role="alert" data-variant="success">
  <strong>Success!</strong> Changes saved.
</div>

<div role="alert" data-variant="warning">
  <strong>Warning!</strong> Review before continuing.
</div>

<div role="alert" data-variant="error">
  <strong>Error!</strong> Something went wrong.
</div>
```

#### Cards

Use `className="card"` for visual box styling.

```jsx
<article className="card">
  <header>
    <h3>Card Title</h3>
    <p>Card description.</p>
  </header>
  <p>Card content here.</p>
  <footer className="flex gap-2 mt-4">
    <button className="outline">Cancel</button>
    <button>Save</button>
  </footer>
</article>
```

#### Dialogs

Use `<dialog>` with `commandfor` and `command` attributes.

```jsx
<button commandFor="myDialog" command="show-modal">
  Open Dialog
</button>

<dialog id="myDialog" closedby="any">
  <form method="dialog">
    <header>
      <h3>Title</h3>
      <p>Dialog description.</p>
    </header>
    <div>
      <p>Dialog content here.</p>
    </div>
    <footer>
      <button
        type="button"
        commandFor="myDialog"
        command="close"
        className="outline"
      >
        Cancel
      </button>
      <button value="confirm">Confirm</button>
    </footer>
  </form>
</dialog>
```

#### Dropdowns (WebComponent)

Wrap in `<ot-dropdown>`. Use `popovertarget` on trigger, `popover` on menu.

```jsx
<ot-dropdown>
  <button popovertarget="myMenu" className="outline">
    Options
    <svg viewBox="0 0 24 24" width="16" height="16">
      <path d="m6 9 6 6 6-6" />
    </svg>
  </button>

  <menu popover id="myMenu">
    <button role="menuitem">Profile</button>
    <button role="menuitem">Settings</button>
    <hr />
    <button role="menuitem">Logout</button>
  </menu>
</ot-dropdown>
```

#### Tabs (WebComponent)

Wrap in `<ot-tabs>`. Use `role="tablist"`, `role="tab"`, `role="tabpanel"`.

```jsx
<ot-tabs>
  <div role="tablist">
    <button role="tab">Account</button>
    <button role="tab">Password</button>
    <button role="tab">Notifications</button>
  </div>

  <div role="tabpanel">
    <h3>Account Settings</h3>
    <p>Manage account info here.</p>
  </div>

  <div role="tabpanel">
    <h3>Password Settings</h3>
    <p>Change password here.</p>
  </div>

  <div role="tabpanel">
    <h3>Notification Settings</h3>
    <p>Configure preferences here.</p>
  </div>
</ot-tabs>
```

#### Tables

Use `<thead>` and `<tbody>`. Tables auto-styled.

```jsx
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>Role</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Alice Johnson</td>
      <td>alice@example.com</td>
      <td>Admin</td>
      <td>
        <span className="badge success">Active</span>
      </td>
    </tr>
    <tr>
      <td>Bob Smith</td>
      <td>bob@example.com</td>
      <td>Editor</td>
      <td>
        <span className="badge">Active</span>
      </td>
    </tr>
  </tbody>
</table>
```

#### Progress & Spinners

```jsx
// Progress bar
<progress value="60" max="100" />

// Spinner (loading indicator)
<div aria-busy="true" data-spinner="small"></div>
<div aria-busy="true"></div>
<div aria-busy="true" data-spinner="large"></div>

// Spinner in button
<button aria-busy="true" data-spinner="small" disabled>
  Loading
</button>
```

#### Skeleton (Loading Placeholders)

```jsx
<div role="status" className="skeleton line"></div>
<div role="status" className="skeleton box"></div>
```

#### Tooltips

Use `title` attribute on any element.

```jsx
<button title="Save your changes">Save</button>
<button title="Delete this item" data-variant="danger">
  Delete
</button>
```

#### Sidebar Layout

```jsx
<body data-sidebar-layout>
  <header data-topnav>
    <button data-sidebar-toggle>Menu</button>
    <nav><!-- top nav items --></nav>
  </header>

  <aside data-sidebar>
    <nav><!-- sidebar items --></nav>
  </aside>

  <main>
    <!-- main content -->
  </main>
</body>
```

### OAT Best Practices

1. **Use semantic HTML**: Leverage native elements (`<button>`, `<input>`, `<form>`, etc.)
2. **Minimal classes**: OAT relies on data attributes and semantic tags
3. **Accessibility built-in**: ARIA attributes and keyboard navigation work out-of-box
4. **No JavaScript required**: Most components work without JS; WebComponents for interactive features
5. **Utility classes**: See [utilities.css](https://github.com/knadh/oat/blob/master/src/css/utilities.css) for layout helpers

---

## Part 4: Folder Structure Conventions

Organize code for scalability, clarity, and maintainability.

### Root Level Structure

```
project-root/
├── src/                    # All source code
├── public/                 # Static assets
├── docs/                   # Documentation
├── tests/                  # Test files (or co-locate with source)
├── build/                  # Build output
├── package.json
├── tsconfig.json
└── biome.jsonc            # Biome config
```

### `src/` Directory Organization

```
src/
├── components/             # React components
│   ├── common/            # Reusable components (Button, Card, etc.)
│   ├── features/          # Feature-specific components
│   └── layouts/           # Layout components
├── pages/                 # Page-level components (if using routing)
├── hooks/                 # Custom React hooks
├── utils/                 # Utility functions
├── types/                 # TypeScript types/interfaces
├── styles/                # Global styles, Tailwind configs
├── lib/                   # Library code (adapters, services)
├── constants/             # App-wide constants
└── app.tsx                # Root component
```

### Components Folder Structure

```
components/
├── Button/
│   ├── Button.tsx         # Component code
│   ├── Button.test.tsx    # Component tests
│   └── index.ts           # Export (avoid barrel pattern)
├── Card/
│   ├── Card.tsx
│   ├── Card.test.tsx
│   └── index.ts
└── DialogForm/            # Compound component
    ├── Dialog.tsx
    ├── DialogContent.tsx
    ├── DialogActions.tsx
    ├── index.ts
    └── DialogForm.test.tsx
```

**Key Rules**:
- **One component per file** (except compound components)
- **Co-locate tests** next to components
- **Avoid index files** that re-export everything (barrel files)
- **Use named exports** for components

### Utils & Hooks Folder

```
utils/
├── string.ts              # String utilities
├── number.ts              # Number utilities
├── date.ts                # Date utilities
└── formatting.ts          # Formatting utilities

hooks/
├── useForm.ts             # Custom hook
├── useLocalStorage.ts
├── useFetch.ts
└── useDimensions.ts
```

### Types Folder

```
types/
├── index.ts               # All type exports
├── api.ts                 # API response types
├── user.ts                # User-related types
└── common.ts              # Common types
```

### Feature-Based Organization (Alternative for Large Apps)

When using OAT + React for larger apps:

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── utils/
│   │   ├── types/
│   │   └── auth.module.ts
│   ├── dashboard/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── types/
│   └── settings/
│       └── ...
├── shared/                # Truly shared resources
│   ├── components/
│   ├── hooks/
│   └── utils/
└── app.tsx
```

### Naming Conventions

- **Components**: PascalCase (`UserProfile.tsx`, `SubmitButton.tsx`)
- **Hooks**: camelCase with `use` prefix (`useForm.ts`, `useWindowSize.ts`)
- **Utilities**: camelCase (`formatDate.ts`, `calculateTotal.ts`)
- **Types**: PascalCase (`User`, `ApiResponse`, `FormConfig`)
- **Constants**: UPPER_SNAKE_CASE (`API_BASE_URL`, `MAX_RETRIES`)
- **Folders**: kebab-case (`user-profile`, `form-builder`)

### Import/Export Patterns

```typescript
// ✓ Good: Direct imports, avoid barrels
import { Button } from '@/components/Button';
import { useForm } from '@/hooks/useForm';
import type { User } from '@/types/user';

// ✗ Bad: Barrel imports
import { Button, Card, Dialog } from '@/components';
```

### Managing Styles with OAT

Since OAT provides semantic styling:

```
styles/
├── global.css             # Global reset, OAT setup
├── variables.css          # CSS custom properties
└── overrides.css          # Project-specific overrides
```

```css
/* global.css */
@import 'https://cdn.jsdelivr.net/npm/oat@latest/dist/oat.min.css';

/* Project customizations */
:root {
  --color-primary: #007bff;
  --color-secondary: #6c757d;
}
```

---

## Testing Standards

- Write assertions inside `it()` or `test()` blocks
- Avoid done callbacks in async tests - use async/await instead
- Don't use `.only` or `.skip` in committed code
- Keep test suites reasonably flat - avoid excessive `describe` nesting
- Co-locate tests with components in the same folder
- Use descriptive test names that explain expected behavior

---

## When Biome Can't Help

Biome's linter will catch most issues automatically. Focus your attention on:

1. **Business logic correctness** - Biome can't validate your algorithms
2. **Meaningful naming** - Use descriptive names for functions, variables, and types
3. **Architecture decisions** - Component structure, data flow, and API design
4. **Edge cases** - Handle boundary conditions and error states
5. **User experience** - Accessibility, performance, and usability considerations
6. **Documentation** - Add comments for complex logic, but prefer self-documenting code

---

## Final Checklist Before Committing

- [ ] Run `bun x ultracite fix` to auto-format code
- [ ] Run `bun x ultracite check` to verify no lint errors
- [ ] Verify components follow OAT patterns (semantic HTML, proper roles)
- [ ] Check React performance best practices (no waterfalls, memoization, etc.)
- [ ] Verify folder structure follows conventions
- [ ] Remove all `console.log`, `debugger`, `alert` statements
- [ ] Add tests for new components and utilities
- [ ] Verify types are explicit and meaningful
- [ ] Check accessibility (semantic HTML, ARIA, keyboard navigation)

Most formatting and common issues are automatically fixed by Biome. Run `bun x ultracite fix` before committing to ensure compliance.


# Ultracite Code Standards

This project uses **Ultracite**, a zero-config preset that enforces strict code quality standards through automated formatting and linting.

## Quick Reference

- **Format code**: `bun x ultracite fix`
- **Check for issues**: `bun x ultracite check`
- **Diagnose setup**: `bun x ultracite doctor`

Biome (the underlying engine) provides robust linting and formatting. Most issues are automatically fixable.

---

## Core Principles

Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

### Type Safety & Explicitness

- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

### Modern JavaScript/TypeScript

- Use arrow functions for callbacks and short functions
- Prefer `for...of` loops over `.forEach()` and indexed `for` loops
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safer property access
- Prefer template literals over string concatenation
- Use destructuring for object and array assignments
- Use `const` by default, `let` only when reassignment is needed, never `var`

### Async & Promises

- Always `await` promises in async functions - don't forget to use the return value
- Use `async/await` syntax instead of promise chains for better readability
- Handle errors appropriately in async code with try-catch blocks
- Don't use async functions as Promise executors

### React & JSX

- Use function components over class components
- Call hooks at the top level only, never conditionally
- Specify all dependencies in hook dependency arrays correctly
- Use the `key` prop for elements in iterables (prefer unique IDs over array indices)
- Nest children between opening and closing tags instead of passing as props
- Don't define components inside other components
- Use semantic HTML and ARIA attributes for accessibility:
  - Provide meaningful alt text for images
  - Use proper heading hierarchy
  - Add labels for form inputs
  - Include keyboard event handlers alongside mouse events
  - Use semantic elements (`<button>`, `<nav>`, etc.) instead of divs with roles

### Error Handling & Debugging

- Remove `console.log`, `debugger`, and `alert` statements from production code
- Throw `Error` objects with descriptive messages, not strings or other values
- Use `try-catch` blocks meaningfully - don't catch errors just to rethrow them
- Prefer early returns over nested conditionals for error cases

### Code Organization

- Keep functions focused and under reasonable cognitive complexity limits
- Extract complex conditions into well-named boolean variables
- Use early returns to reduce nesting
- Prefer simple conditionals over nested ternary operators
- Group related code together and separate concerns

### Security

- Add `rel="noopener"` when using `target="_blank"` on links
- Avoid `dangerouslySetInnerHTML` unless absolutely necessary
- Don't use `eval()` or assign directly to `document.cookie`
- Validate and sanitize user input

### Performance

- Avoid spread syntax in accumulators within loops
- Use top-level regex literals instead of creating them in loops
- Prefer specific imports over namespace imports
- Avoid barrel files (index files that re-export everything)
- Use proper image components (e.g., Next.js `<Image>`) over `<img>` tags

### Framework-Specific Guidance

**Next.js:**
- Use Next.js `<Image>` component for images
- Use `next/head` or App Router metadata API for head elements
- Use Server Components for async data fetching instead of async Client Components

**React 19+:**
- Use ref as a prop instead of `React.forwardRef`

**Solid/Svelte/Vue/Qwik:**
- Use `class` and `for` attributes (not `className` or `htmlFor`)

---

## Testing

- Write assertions inside `it()` or `test()` blocks
- Avoid done callbacks in async tests - use async/await instead
- Don't use `.only` or `.skip` in committed code
- Keep test suites reasonably flat - avoid excessive `describe` nesting

## When Biome Can't Help

Biome's linter will catch most issues automatically. Focus your attention on:

1. **Business logic correctness** - Biome can't validate your algorithms
2. **Meaningful naming** - Use descriptive names for functions, variables, and types
3. **Architecture decisions** - Component structure, data flow, and API design
4. **Edge cases** - Handle boundary conditions and error states
5. **User experience** - Accessibility, performance, and usability considerations
6. **Documentation** - Add comments for complex logic, but prefer self-documenting code

---

Most formatting and common issues are automatically fixed by Biome. Run `bun x ultracite fix` before committing to ensure compliance.

---
> Source: [khanhthanhdev/electrobun-rms](https://github.com/khanhthanhdev/electrobun-rms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
