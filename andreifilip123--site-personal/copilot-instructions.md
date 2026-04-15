## site-personal

> The application uses the experimental View Transitions API from Next.js to create smooth transitions between pages.

# View Transitions Guide

The application uses the experimental View Transitions API from Next.js to create smooth transitions between pages.

## Implementation Details

- View transitions are implemented using the `ViewTransition` component from Next.js
- Each section has multiple named transitions:
  - Page transition (e.g., `code-page`)
  - Photo transition (e.g., `code-photo`)
  - Title transition (e.g., `code-title`)

## Example Usage

In [app/page.tsx](mdc:app/page.tsx), transitions are wrapped around elements that should animate:

```tsx
<ViewTransition name="code-page">
  <Link href="/code">
    <ViewTransition name="code-photo">
      {/* Photo content */}
    </ViewTransition>
    <ViewTransition name="code-title">
      {/* Title content */}
    </ViewTransition>
  </Link>
</ViewTransition>
```

The transitions are coordinated between pages to create smooth animations when navigating between different sections.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreifilip123)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andreifilip123)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
