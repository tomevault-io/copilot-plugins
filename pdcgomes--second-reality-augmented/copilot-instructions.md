## editor-react

> React component patterns and state management for the editor


# Editor React Conventions

## Components

- Functional components only (React 18)
- One component per file, named export matching filename

## State Management

- Zustand for all shared state (playhead, selection, zoom, project data)
- No prop-drilling — access store directly via hooks in any component
- Local UI state (hover, drag) can use `useState`/`useRef`

## Canvas Panels

Timeline, Tracker, and Preview use React refs for lifecycle management but raw canvas APIs for drawing:

```jsx
const canvasRef = useRef(null);
useEffect(() => {
  const ctx = canvasRef.current.getContext('2d');
  // draw loop here
}, []);
```

## Styling

- Tailwind utility classes — dark theme with FL Studio aesthetic
- shadcn/ui for interactive primitives (sliders, dropdowns, tooltips)
- No custom CSS files unless absolutely necessary

---
> Source: [pdcgomes/second-reality-augmented](https://github.com/pdcgomes/second-reality-augmented) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
