## portfolio

> Enforce using named CSS classes with @apply instead of inline Tailwind utilities in markup


# Use Named CSS Classes Instead of Inline Tailwind Utilities

Define styles as named classes using `@apply`, then reference those class names in markup.

## Pattern

**CSS file**
```css
@layer components {
  .contact-link {
    @apply bg-neutral-black text-white w-fit rounded-2xl px-4 py-2 no-underline hover:opacity-90 transition-opacity inline-flex items-center gap-2;
  }
}
```

**Markup**
```html
<!-- ✅ GOOD -->
<a class="contact-link" href="/contact">Get in touch</a>

<!-- ❌ BAD -->
<a class="bg-neutral-black text-white w-fit rounded-2xl px-4 py-2 no-underline hover:opacity-90 transition-opacity inline-flex items-center gap-2" href="/contact">Get in touch</a>
```

## Rules

- Place reusable component styles in `@layer components` and one-off utilities in `@layer utilities`
- Name classes semantically after their role (e.g. `.submit-button`, `.card`, `.contact-link`), not their appearance
- Inline Tailwind utilities in markup are only acceptable for truly one-off, single-property overrides that won't be reused

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/loqimean)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/loqimean)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
