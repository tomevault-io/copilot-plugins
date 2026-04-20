## candidateranker

> Web frontend development rules


# Web Frontend Development Rules

## Component Patterns
- One component per file — file name matches component name
- Prefer functional components over class components
- Keep components focused — if it does too many things, split it
- Extract shared logic into custom hooks (React) or composables (Vue)
- Co-locate component styles, tests, and types with the component

## State Management
- Start with local state — only lift when needed
- Use the framework's built-in state management first (React Context, Vue's reactive)
- Add external state management (Zustand, Pinia, Redux) only when local state becomes unwieldy
- Keep state as close to where it's used as possible
- Derive values from state instead of storing duplicate data

## TypeScript
- Use strict TypeScript configuration (`"strict": true`)
- Define explicit types for component props
- Avoid `any` — use `unknown` when the type is genuinely unknown, then narrow
- Use discriminated unions for complex state
- Export types alongside their implementations
- Define API response types that match the backend contracts

## Accessibility (a11y)
- Use semantic HTML elements (`<button>`, `<nav>`, `<main>`, `<article>`)
- All interactive elements must be keyboard accessible
- Images must have `alt` attributes (empty `alt=""` for decorative images)
- Form inputs must have associated `<label>` elements
- Use ARIA attributes only when semantic HTML isn't sufficient
- Maintain visible focus indicators — don't remove outlines without replacing them
- Test with keyboard navigation
- Ensure color contrast meets WCAG 2.1 AA (4.5:1 for text, 3:1 for large text)
- Manage focus on route changes and modal open/close

## Responsive Design
- Use mobile-first approach — base styles for mobile, `min-width` media queries for larger
- Use relative units (`rem`, `em`, `%`) over fixed pixels for typography and spacing
- Use CSS Grid for page layouts, Flexbox for component layouts
- Test at standard breakpoints: 320px, 768px, 1024px, 1440px
- Don't hide critical content on mobile — restructure instead

## Performance
- Lazy load routes and heavy components
- Optimize images — use modern formats (WebP, AVIF), appropriate sizes, loading="lazy"
- Minimize bundle size — check with bundle analyzer, avoid importing entire libraries
- Memoize expensive computations and callbacks (useMemo, useCallback)
- Use virtualization for long lists (1000+ items)
- Debounce user input that triggers expensive operations (search, validation)
- Set performance budgets: initial JS < 200KB, LCP < 2.5s, FID < 100ms

## XSS Prevention
- Never use `dangerouslySetInnerHTML` (React) or `v-html` (Vue) with user content
- Sanitize any HTML that must be rendered from external sources (use DOMPurify)
- Use Content Security Policy headers
- Escape user input in URL parameters
- Don't trust data from URL query parameters, localStorage, or cookies without validation

## Forms
- Validate on both client and server — client for UX, server for security
- Show validation errors inline, next to the relevant field
- Disable submit button during submission to prevent double-submit
- Handle loading, error, and success states explicitly
- Use controlled components for form inputs
- Preserve form data on validation errors — don't clear the form

## Error Handling & Observability
- Use error boundaries (React) to catch rendering errors
- Show user-friendly error messages — not stack traces
- Provide retry actions for failed network requests
- Log errors to a monitoring service (not just console.log)
- Handle loading states — never show a blank screen while loading
- Track: page load time, API call duration, error rate, user interactions
- Use structured error reporting: error type, component, user action, stack trace

## Testing
- Use Vitest or Jest for unit tests, Playwright or Cypress for e2e
- Test component rendering, user interactions, and edge cases
- Test accessibility with axe-core or similar automated tools
- Write tests alongside implementation
- Mock API calls in component tests

## Common Mistakes to Avoid
- Don't store derived data in state — compute it from source state
- Don't fetch data in rendering logic — use effects or data fetching libraries
- Don't ignore the dependency array in useEffect — it causes stale closures
- Don't create inline objects/arrays in JSX props — they trigger unnecessary re-renders
- Don't use index as key for lists that can reorder — use stable unique IDs
- Don't block rendering with synchronous operations — use async patterns
- Don't use `console.log` for production error tracking — use a proper error service
- Don't ignore TypeScript errors with `@ts-ignore` — fix the types

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TGabeCar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
