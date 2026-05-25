## agents-md

> This guide provides a comprehensive overview of the Remix Component API, its runtime behavior, and practical use cases for building interactive UIs.

# Remix Component - Agent Guide

This guide provides a comprehensive overview of the Remix Component API, its runtime behavior, and practical use cases for building interactive UIs.

## Getting Started

### Creating a Root

To start using Remix Component, create a root and render your top-level component:

```tsx
import { createRoot } from '@remix-run/component'
import type { Handle } from '@remix-run/component'

function App(handle: Handle) {
  return () => (
    <div>
      <h1>Hello, World!</h1>
    </div>
  )
}

// Create a root attached to a DOM element
let container = document.getElementById('app')!
let root = createRoot(container)

// Render your app
root.render(<App />)
```

The `createRoot` function takes a DOM element (or `document.body`) and returns a root object with a `render` method. You can call `render` multiple times to update the app:

{% raw %}
```tsx
function App(handle: Handle) {
  let count = 0

  return () => (
    <div>
      <h1>Count: {count}</h1>
      <button
        on={{
          click() {
            count++
            handle.update()
          },
        }}
      >
        Increment
      </button>
    </div>
  )
}

let root = createRoot(document.body)
root.render(<App />)

// Later, you can update the app by calling render again
// root.render(<App />)
```
{% endraw %}

### Root Methods

The root object provides several methods:

- **`render(node)`** - Renders a component tree into the root container
- **`flush()`** - Synchronously flushes all pending updates and tasks
- **`remove()`** - Removes the component tree and cleans up

```tsx
let root = createRoot(document.body)

// Render initial app
root.render(<App />)

// Flush any pending updates synchronously
root.flush()

// Later, remove the app
root.remove()
```

## Component Factory and Runtime Behavior

### Component Structure

All components follow a consistent two-phase structure:

1. **Setup Phase** - Runs once when the component is first created
2. **Render Phase** - Runs on initial render and every update afterward

```tsx
function MyComponent(handle: Handle, setup: SetupType) {
  // Setup phase: runs once
  let state = initializeState(setup)

  // Return render function: runs on every update
  return (props: Props) => {
    return <div>{/* render content */}</div>
  }
}
```

### Runtime Behavior

When a component is rendered:

1. **First Render**:

   - The component function is called with `handle` and the `setup` prop
   - The returned render function is stored
   - The render function is called with regular props
   - Any tasks queued via `handle.queueTask()` are executed after rendering

2. **Subsequent Updates**:

   - Only the render function is called
   - Setup phase is skipped, setup closure persists for the lifetime of the component instance
   - Props are passed to the render function
   - The `setup` prop is stripped from props
   - Tasks queued during the update are executed after rendering

3. **Component Removal**:
   - `handle.signal` is aborted
   - All event listeners registered via `handle.on()` are automatically cleaned up
   - Any queued tasks are executed with an aborted signal

### Setup vs Props

The `setup` prop is special - it's only available in the setup phase and is automatically excluded from props. This prevents accidental stale captures:

```tsx
function Counter(handle: Handle, setup: number) {
  // setup prop (e.g., initialCount) only available here
  let count = setup

  return (props: { label: string }) => {
    // props only receives { label } - setup is excluded
    return (
      <div>
        {props.label}: {count}
      </div>
    )
  }
}

// Usage
let element = <Counter setup={10} label="Count" />
```

## Handle API

The `Handle` object provides the component's interface to the framework:

### `handle.update(task?)`

Schedules a component update. Optionally accepts a task to run after the update completes.

```tsx
function Counter(handle: Handle) {
  let count = 0

  return () => (
    <button
      on={{
        click() {
          count++
          handle.update()
        },
      }}
    >
      Count: {count}
    </button>
  )
}
```

With a task:

```tsx
function Player(handle: Handle) {
  let isPlaying = false
  let stopButton: HTMLButtonElement

  return () => (
    <button
      disabled={isPlaying}
      on={{
        click() {
          isPlaying = true
          handle.update(() => {
            // Task runs after update completes
            stopButton.focus()
          })
        },
      }}
    >
      Play
    </button>
  )
}
```

### `handle.queueTask(task)`

Schedules a task to run after the next update. The task receives an `AbortSignal` that's aborted when:

- The component re-renders (new render cycle starts)
- The component is removed from the tree

**Use `queueTask` in event handlers when work needs to happen after DOM changes:**

```tsx
function Form(handle: Handle) {
  let showDetails = false
  let detailsSection: HTMLElement

  return () => (
    <form>
      <input
        type="checkbox"
        checked={showDetails}
        on={{
          change(event) {
            showDetails = event.currentTarget.checked
            handle.update()
            if (showDetails) {
              // Queue DOM operation after the new section renders
              handle.queueTask(() => {
                detailsSection.scrollIntoView({ behavior: 'smooth' })
              })
            }
          },
        }}
      />
      {showDetails && (
        <section connect={(node) => (detailsSection = node)}>Details content</section>
      )}
    </form>
  )
}
```

**Use `queueTask` for work that needs to be reactive to prop changes:**

When you need to perform async work (like data fetching) that should respond to prop changes, use `queueTask` in the render function. The signal will be aborted if props change or the component is removed from the tree:

{% raw %}
```tsx
function DataFetcher(handle: Handle) {
  let data: any = null
  let error: string | null = null

  return (props: { id: string }) => {
    // Fetch when props change
    handle.queueTask((signal) => {
      fetch(`/api/data/${props.id}`, { signal })
        .then((res) => res.json())
        .then((result) => {
          if (!signal.aborted) {
            data = result
            error = null
            handle.update()
          }
        })
        .catch((err) => {
          if (!signal.aborted) {
            error = err.message
            handle.update()
          }
        })
    })

    return (
      <div>
        {error && <div>Error: {error}</div>}
        {data && <div>{JSON.stringify(data)}</div>}
      </div>
    )
  }
}
```
{% endraw %}

**❌ Anti-pattern: Don't create states as values to "react to" on the next render with `queueTask`:**

```tsx
// ❌ Avoid: Creating state just to react to it in queueTask
function BadExample(handle: Handle) {
  let shouldLoad = false // Unnecessary state

  return () => (
    <div>
      <button
        on={{
          click() {
            shouldLoad = true // Setting state just to trigger queueTask
            handle.update()
            handle.queueTask(() => {
              if (shouldLoad) {
                // Do work
              }
            })
          },
        }}
      >
        Load
      </button>
    </div>
  )
}

// ✅ Prefer: Do the work directly in the event handler or queueTask
function GoodExample(handle: Handle) {
  return () => (
    <div>
      <button
        on={{
          click() {
            handle.queueTask(() => {
              // Do work directly - no intermediate state needed
            })
          },
        }}
      >
        Load
      </button>
    </div>
  )
}
```

**Signals in events and tasks are how you manage interruptions and disconnects:**

Both event handlers and `queueTask` receive `AbortSignal` parameters that are automatically aborted when:

- The component is removed from the tree
- For event handlers: The handler is re-entered (user triggers another event)
- For `queueTask`: The component re-renders (props changed)

Always check `signal.aborted` or pass the signal to async APIs (like `fetch`) to handle interruptions gracefully.

### `handle.signal`

An `AbortSignal` that's aborted when the component is disconnected. Useful for cleanup operations.

```tsx
function Clock(handle: Handle) {
  let interval = setInterval(() => {
    if (handle.signal.aborted) {
      clearInterval(interval)
      return
    }
    handle.update()
  }, 1000)

  return () => <span>{new Date().toString()}</span>
}
```

Or using event listeners:

```tsx
function Clock(handle: Handle) {
  let interval = setInterval(handle.update, 1000)
  handle.signal.addEventListener('abort', () => clearInterval(interval))

  return () => <span>{new Date().toString()}</span>
}
```

### `handle.on(target, listeners)`

Listen to an `EventTarget` with automatic cleanup when the component disconnects. Ideal for global event targets like `document` and `window`.

```tsx
function KeyboardTracker(handle: Handle) {
  let keys: string[] = []

  handle.on(document, {
    keydown(event) {
      keys.push(event.key)
      handle.update()
    },
  })

  return () => <div>Keys: {keys.join(', ')}</div>
}
```

### `handle.id`

Stable identifier per component instance. Useful for HTML APIs like `htmlFor`, `aria-owns`, etc.

```tsx
function LabeledInput(handle: Handle) {
  return () => (
    <div>
      <label htmlFor={handle.id}>Name</label>
      <input id={handle.id} type="text" />
    </div>
  )
}
```

### `handle.context`

Context API for ancestor/descendant communication. Use `handle.context.set()` to provide values and `handle.context.get()` to consume them.

**Important:** `handle.context.set()` does not cause any updates - it simply stores a value. If you need the component tree to update when context changes, call `handle.update()` after setting the context.

```tsx
function App(handle: Handle<{ theme: string }>) {
  handle.context.set({ theme: 'dark' })

  return () => (
    <div>
      <Header />
      <Content />
    </div>
  )
}

function Header(handle: Handle) {
  let { theme } = handle.context.get(App)
  return () => <header css={{ backgroundColor: theme === 'dark' ? '#000' : '#fff' }}>Header</header>
}
```

## Rendering and Composition

### Basic Rendering

The simplest component just returns JSX:

```tsx
function Greeting() {
  return (props: { name: string }) => <div>Hello, {props.name}!</div>
}

let el = <Greeting name="World" />
```

### Prop Passing

Props flow from parent to child through JSX attributes:

```tsx
function Parent() {
  return () => <Child message="Hello from parent" count={42} />
}

function Child() {
  return (props: { message: string; count: number }) => (
    <div>
      <p>{props.message}</p>
      <p>Count: {props.count}</p>
    </div>
  )
}
```

### Stateful Updates

State is managed with plain JavaScript variables. Call `handle.update()` to trigger a re-render:

```tsx
function Counter(handle: Handle) {
  let count = 0

  return () => (
    <div>
      <span>Count: {count}</span>
      <button
        on={{
          click() {
            count++
            handle.update()
          },
        }}
      >
        Increment
      </button>
    </div>
  )
}
```

### State Management Best Practices

#### Use Minimal Component State

Only store state that's needed for rendering. Derive computed values instead of storing them, and avoid storing input state that you don't need.

**Derive computed values:**

```tsx
// ❌ Avoid: Storing computed values
function TodoList(handle: Handle) {
  let todos: string[] = []
  let completedCount = 0 // Unnecessary state

  return () => (
    <div>
      {todos.map((todo, i) => (
        <div key={i}>{todo}</div>
      ))}
      <div>Completed: {completedCount}</div>
    </div>
  )
}

// ✅ Prefer: Derive computed values in render
function TodoList(handle: Handle) {
  let todos: Array<{ text: string; completed: boolean }> = []

  return () => {
    // Derive computed value in render
    let completedCount = todos.filter((t) => t.completed).length

    return (
      <div>
        {todos.map((todo, i) => (
          <div key={i}>{todo.text}</div>
        ))}
        <div>Completed: {completedCount}</div>
      </div>
    )
  }
}
```

**Don't store input state you don't need:**

```tsx
// ❌ Avoid: Storing input value when you only need it on submit
function SearchForm(handle: Handle) {
  let query = '' // Unnecessary state

  return () => (
    <form
      on={{
        submit(event) {
          event.preventDefault()
          let formData = new FormData(event.currentTarget)
          let query = formData.get('query') as string
          // Use query for search
        },
      }}
    >
      <input name="query" />
      <button type="submit">Search</button>
    </form>
  )
}

// ✅ Prefer: Read input value directly from the form
function SearchForm(handle: Handle) {
  return () => (
    <form
      on={{
        submit(event) {
          event.preventDefault()
          let formData = new FormData(event.currentTarget)
          let query = formData.get('query') as string
          // Use query for search - no component state needed
        },
      }}
    >
      <input name="query" />
      <button type="submit">Search</button>
    </form>
  )
}
```

#### Do Work in Event Handlers

Do as much work as possible in event handlers with minimal component state. Use the event handler scope for transient event state, and only capture to component state if it's used for rendering.

**Use event handler scope for transient state:**

```tsx
// ❌ Avoid: Storing transient state in component
function FormValidator(handle: Handle) {
  let validationError: string | null = null // Only needed during validation

  return () => (
    <form
      on={{
        submit(event) {
          event.preventDefault()
          let formData = new FormData(event.currentTarget)
          let email = formData.get('email') as string

          // Validation logic
          if (!email.includes('@')) {
            validationError = 'Invalid email'
            handle.update()
            return
          }

          // Submit form
          validationError = null
          handle.update()
        },
      }}
    >
      {validationError && <div>{validationError}</div>}
      <input name="email" />
      <button type="submit">Submit</button>
    </form>
  )
}

// ✅ Prefer: Keep transient state in event handler scope
function FormValidator(handle: Handle) {
  let validationError: string | null = null // Only stored if needed for rendering

  return () => (
    <form
      on={{
        submit(event) {
          event.preventDefault()
          let formData = new FormData(event.currentTarget)
          let email = formData.get('email') as string

          // Validation logic - keep transient state in handler scope
          if (!email.includes('@')) {
            validationError = 'Invalid email' // Only store if rendering needs it
            handle.update()
            return
          }

          // Submit form - clear error if it exists
          if (validationError) {
            validationError = null
            handle.update()
          }
        },
      }}
    >
      {validationError && <div>{validationError}</div>}
      <input name="email" />
      <button type="submit">Submit</button>
    </form>
  )
}
```

**Only store state needed for rendering:**

```tsx
// ✅ Good: Store state that affects rendering
function Toggle(handle: Handle) {
  let isOpen = false // Needed for rendering conditional content

  return () => (
    <div>
      <button
        on={{
          click() {
            isOpen = !isOpen
            handle.update()
          },
        }}
      >
        Toggle
      </button>
      {isOpen && <div>Content</div>}
    </div>
  )
}

// ✅ Good: Do work in handler, only store what renders need
function SearchResults(handle: Handle) {
  let results: string[] = [] // Needed for rendering
  let loading = false // Needed for rendering loading state

  return () => (
    <div>
      <input
        on={{
          async input(event, signal) {
            let query = event.currentTarget.value
            // Do work in handler scope
            loading = true
            handle.update()

            let response = await fetch(`/search?q=${query}`, { signal })
            let data = await response.json()
            if (signal.aborted) return

            // Only store what's needed for rendering
            results = data.results
            loading = false
            handle.update()
          },
        }}
      />
      {loading && <div>Loading...</div>}
      {results.map((result, i) => (
        <div key={i}>{result}</div>
      ))}
    </div>
  )
}
```

### CSS Prop with Pseudo-Selectors and Descendant Selectors

The `css` prop provides inline styling with support for pseudo-selectors, pseudo-elements, attribute selectors, descendant selectors, and media queries. It follows modern CSS nesting selector rules.

#### Basic CSS Prop

```tsx
function Button() {
  return () => (
    <button
      css={{
        color: 'white',
        backgroundColor: 'blue',
        padding: '12px 24px',
        borderRadius: '4px',
        border: 'none',
        cursor: 'pointer',
      }}
    >
      Click me
    </button>
  )
}
```

#### Performance: CSS Prop vs Style Prop

The `css` prop produces static styles that are inserted into the document as CSS rules, while the `style` prop applies styles directly to the element. For **dynamic styles** that change frequently**, use the `style` prop instead to avoid unnecessary CSS rule generation.

```tsx
// ❌ Avoid: Using css prop for dynamic styles
function ProgressBar(handle: Handle) {
  let progress = 0

  return () => (
    <div
      css={{
        width: `${progress}%`, // Creates new CSS rule on every update
        backgroundColor: 'blue',
      }}
    >
      {progress}%
    </div>
  )
}

// ✅ Prefer: Using style prop for dynamic styles
function ProgressBar(handle: Handle) {
  let progress = 0

  return () => (
    <div
      css={{
        backgroundColor: 'blue', // Static styles in css prop
      }}
      style={{
        width: `${progress}%`, // Dynamic styles in style prop
      }}
    >
      {progress}%
    </div>
  )
}
```

**Use the `css` prop for:**

- Static styles that don't change
- Styles that need pseudo-selectors (`:hover`, `:focus`, etc.)
- Styles that need media queries

**Use the `style` prop for:**

- Dynamic styles that change based on state or props
- Computed values that update frequently

#### Pseudo-Selectors

Use `&` to reference the current element in pseudo-selectors:

```tsx
function Button() {
  return () => (
    <button
      css={{
        color: 'white',
        backgroundColor: 'blue',
        padding: '12px 24px',
        borderRadius: '4px',
        border: 'none',
        cursor: 'pointer',
        '&:hover': {
          backgroundColor: 'darkblue',
          transform: 'translateY(-1px)',
        },
        '&:active': {
          backgroundColor: 'navy',
          transform: 'translateY(0)',
        },
        '&:focus': {
          outline: '2px solid yellow',
          outlineOffset: '2px',
        },
        '&:disabled': {
          opacity: 0.5,
          cursor: 'not-allowed',
        },
      }}
    >
      Click me
    </button>
  )
}
```

#### Pseudo-Elements

Use `&::before` and `&::after` for pseudo-elements:

```tsx
function Badge() {
  return (props: { count: number }) => (
    <div
      css={{
        position: 'relative',
        display: 'inline-block',
        '&::before': {
          content: '""',
          position: 'absolute',
          top: '-4px',
          right: '-4px',
          width: '8px',
          height: '8px',
          backgroundColor: 'red',
          borderRadius: '50%',
        },
      }}
    >
      {props.count > 0 && <span>{props.count}</span>}
    </div>
  )
}
```

#### Attribute Selectors

Use `&[attribute]` for attribute selectors:

```tsx
function Input() {
  return (props: { required?: boolean }) => (
    <input
      required={props.required}
      css={{
        padding: '8px',
        border: '1px solid #ccc',
        borderRadius: '4px',
        '&[required]': {
          borderColor: 'red',
        },
        '&[aria-invalid="true"]': {
          borderColor: 'red',
          outline: '2px solid red',
        },
      }}
    />
  )
}
```

#### Descendant Selectors

Use class names or element selectors directly for descendant selectors:

```tsx
function Card() {
  return (props: { children: RemixNode }) => (
    <div
      css={{
        padding: '20px',
        border: '1px solid #ddd',
        borderRadius: '8px',
        backgroundColor: 'white',
        boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
        // Style descendants
        '& h2': {
          marginTop: 0,
          fontSize: '24px',
          fontWeight: 'bold',
        },
        '& p': {
          color: '#666',
          lineHeight: 1.6,
        },
        '& .icon': {
          width: '24px',
          height: '24px',
          marginRight: '8px',
        },
        '& button': {
          marginTop: '16px',
        },
      }}
    >
      {props.children}
    </div>
  )
}
```

#### When to Use Nested Selectors

Use nested selectors when **parent state affects children**. Don't nest when you can style the element directly.

**This is preferable to creating JavaScript state and passing it around.** Instead of managing hover/focus state in JavaScript and passing it as props, use CSS nested selectors to let the browser handle the state declaratively.

**Use nested selectors when:**

1. **Parent state affects children** - Parent hover/focus/state changes child styling (prefer this over JavaScript state management)
2. **Styling descendant elements** - Avoid duplicating styles on every child or creating new components just for styling. Style children from the parent component instead.

**Don't nest when:**

- Styling the element's own pseudo-states (hover, focus, etc.)
- The element controls its own styling

**Example: Parent hover affects children** (use nested selectors, not JavaScript state):

```tsx
// ❌ Avoid: Managing hover state in JavaScript
function CardWithJSState(handle: Handle) {
  let isHovered = false

  return (props: { children: RemixNode }) => (
    <div
      on={{
        mouseenter() {
          isHovered = true
          handle.update()
        },
        mouseleave() {
          isHovered = false
          handle.update()
        },
      }}
      css={{
        border: `1px solid ${isHovered ? 'blue' : '#ddd'}`,
        // ... more conditional styling based on isHovered
      }}
    >
      <div className="title" css={{ color: isHovered ? 'blue' : '#333' }}>
        Title
      </div>
    </div>
  )
}

// ✅ Prefer: CSS nested selectors handle state declaratively
function Card(handle: Handle) {
  return (props: { children: RemixNode }) => (
    <div
      css={{
        border: '1px solid #ddd',
        borderRadius: '8px',
        padding: '20px',
        // Parent hover affects children - use nested selector
        '&:hover': {
          borderColor: 'blue',
          // Child text changes color on parent hover
          '& .title': {
            color: 'blue',
          },
          '& .description': {
            opacity: 1,
          },
        },
        '& .title': {
          fontSize: '20px',
          fontWeight: 'bold',
          color: '#333',
        },
        '& .description': {
          opacity: 0.7,
          marginTop: '8px',
        },
      }}
    >
      <div className="title">Title</div>
    </div>
  )
}
```

**Example: Element's own hover** (style directly, no nesting needed):

```tsx
function Button() {
  return () => (
    <button
      css={{
        backgroundColor: 'blue',
        color: 'white',
        padding: '12px 24px',
        borderRadius: '4px',
        border: 'none',
        cursor: 'pointer',
        // Element's own hover - style directly, no nesting needed
        '&:hover': {
          backgroundColor: 'darkblue',
        },
        '&:active': {
          transform: 'scale(0.98)',
        },
      }}
    >
      Click me
    </button>
  )
}
```

**Example: Navigation with links** (descendant styling is appropriate):

```tsx
function Navigation() {
  return () => (
    <nav
      css={{
        display: 'flex',
        gap: '20px',
        padding: '16px 0',
        borderBottom: '1px solid #ddd',
        // Style all descendant links
        '& a': {
          color: '#0066cc',
          textDecoration: 'none',
          fontSize: '16px',
          fontWeight: 500,
          padding: '8px 12px',
          borderRadius: '4px',
          transition: 'all 0.2s ease',
          '&:hover': {
            backgroundColor: '#f0f0f0',
            color: '#0052a3',
          },
          '&:active': {
            backgroundColor: '#e0e0e0',
          },
        },
        // Style active link differently
        '& a[aria-current="page"]': {
          borderBottom: '2px solid #0066cc',
          color: '#0052a3',
        },
      }}
    >
      <a href="/">Home</a>
      <a href="/about">About</a>
      <a href="/contact">Contact</a>
    </nav>
  )
}
```

## Form Handling

### Controlled Inputs

Controlled inputs manage their value through component state and respond to changes:

{% raw %}
```tsx
function ControlledInput(handle: Handle) {
  let value = ''

  return () => (
    <input
      type="text"
      value={value}
      on={{
        input(event) {
          value = event.currentTarget.value
          handle.update()
        },
      }}
    />
  )
}
```
{% endraw %}

### Form Submission

Handle form submission by reading FormData:

```tsx
function MyForm(handle: Handle) {
  return () => (
    <form
      on={{
        submit(event) {
          event.preventDefault()
          let formData = new FormData(event.currentTarget)
          let values = Object.fromEntries(formData)
          // Handle submission
        },
      }}
    >
      <input name="email" type="email" />
      <input name="password" type="password" />
      <button type="submit">Submit</button>
    </form>
  )
}
```

## Debugging and Best Practices

### When to Use `handle.signal`

Use `handle.signal` in the setup phase to detect when a component will be removed:

```tsx
function DataComponent(handle: Handle) {
  handle.signal.addEventListener('abort', () => {
    // Component is being removed - clean up
  })

  return () => <div>Data</div>
}
```

### Performance Optimization

Minimize re-renders by:

1. **Keeping state local** - Only update state that affects rendering
2. **Using derived values** - Compute values in render, don't store them
3. **Event handler scope** - Do transient work in handlers, don't store it

This architecture makes it easy to reason about updates and prevents unnecessary re-renders.

---
> Source: [ivanm696/remix-remix-3.0.0-alpha.0](https://github.com/ivanm696/remix-remix-3.0.0-alpha.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
