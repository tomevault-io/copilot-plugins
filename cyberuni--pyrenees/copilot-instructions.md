## tailwindcss

> This rule provides guidelines and best practices for using Tailwind CSS in our project. Follow these conventions to maintain consistent styling across the codebase.

# Tailwind CSS Guidelines

This rule provides guidelines and best practices for using Tailwind CSS in our project. Follow these conventions to maintain consistent styling across the codebase.

## Configuration

The main Tailwind configuration is in [tailwind.css](mdc:libraries/design/src/tailwind.css). This file defines our:

- Custom colors
- Extended theme values
- Content paths
- Custom prefix

## Best Practices

1. **Use prefix**
   Our Tailwind classes are prefixed (e.g. `pds:red-100`). Check [tailwind.css](mdc:libraries/design/src/tailwind.css) for the prefix used.

2. **Use Semantic Class Names**
   - Prefer semantic class names in corresponding context.
   - Example: `className="heading-text-sm"` instead of `className:"text-100"`

3. **Class Order**
   Follow this order for classes:
   ```tsx
   className="
     layout-classes    // flex, grid, container
     sizing-classes   // w-, h-, max-w-
     spacing-classes  // p-, m-, gap-
     border-classes   // border-, rounded-
     background      // bg-, gradient-
     typography      // text-, font-
     states          // hover:, focus:, active:
     misc            // cursor-, select-, etc.
   "
   ```

4. **Responsive Design**
   - Use Tailwind's responsive prefixes: `sm:`, `md:`, `lg:`, `xl:`, `2xl:`
   - Desktop-first approach: base styles are for desktop (defaults to `1920px` width), add responsive classes for other screens sizes

   ```tsx
   className="pds:w-full pds:md:w-1/2 pds:lg:w-1/3"
   ```

5. **Dark Mode**
   - Use the `dark:` prefix for dark mode styles
   - Example: `className="pds:bg-white pds:dark:bg-steel-gray-800"`

6. **Handle variants**
   - Use `class-variance-authority` to handle variants

```tsx
// Good
import { cva } from 'class-variance-authority'

const variants = cva([
	'pds:rounded',
	'pds:px-4 pds:py-2'
], {
	variants: {
		appearance: {
			primary: 'pds:bg-blue-500 pds:hover:bg-blue-600',
			secondary: 'pds:bg-gray-500 pds:hover:bg-gray-600'
		}
	},
	defaultVariants: {
		appearance: 'primary'
	}
})

export function Button({ appearance = 'primary', children }) {
	return (
	<button className={variants({ apperance })}>
		{children}
	</button>
)
}
```

7. **Handle tailwind merge**
   When custom `className` is being merged with internal class names,
	 and it is applied to basic Html element or 3rd party component,
	 use the customized `twMerge` function to resolve any conflicts.
	 For example:

```tsx
// Good
import { cva } from 'class-variance-authority'
import { twMerge } from '../utils/tw_merge.ts'

const variants = cva([
	'pds:rounded',
	'pds:px-4 pds:py-2'
], {
	variants: {
		appearance: {
			primary: 'pds:bg-blue-500 pds:hover:bg-blue-600',
			secondary: 'pds:bg-gray-500 pds:hover:bg-gray-600'
		}
	},
	defaultVariants: {
		appearance: 'primary'
	}
})

export function Button({ appearance = 'primary', children }) {
	return (
	<button className={twMerge(variants({ apperance }))}>
		{children}
	</button>
)
}
```

## Common Patterns

### Layout
```tsx
// Centered container
<div className="pds:container pds:mx-auto pds:px-4">

// Card layout
<div className="pds:rounded-lg pds:shadow-md pds:bg-white pds:p-6">

// Grid layout
<div className="pds:grid pds:grid-cols-1 pds:md:grid-cols-2 pds:lg:grid-cols-3 pds:gap-4">
```

### Forms
```tsx
// Input field
<input className="pds:w-full pds:px-3 pds:py-2 pds:border pds:rounded-md pds:focus:ring-2 pds:focus:ring-blue-500 pds:focus:border-blue-500">

// Label
<label className="pds:block pds:text-sm pds:font-medium pds:text-gray-700">
```

### Interactive Elements
```tsx
// Primary button
<button className="pds:px-4 pds:py-2 pds:bg-blue-500 pds:hover:bg-blue-600 pds:text-white pds:vounded-md">

// Secondary button
<button className="pds:px-4 pds:py-2 pds:bg-gray-200 pds:hover:bg-gray-300 pds:text-gray-800 pds:rounded-md">
```

## Resources

- [Tailwind CSS Documentation](mdc:https:/tailwindcss.com/docs)
- [Tailwind CSS Cheat Sheet](mdc:https:/nerdcave.com/tailwind-cheat-sheet)
- [Tailwind UI Components](mdc:https:/tailwindui.com)

## Troubleshooting

1. **Classes not applying:**
   - Check content paths in [tailwind.css](mdc:libraries/design/src/tailwind.css)
   - Verify class names are correct
   - Clear PostCSS cache

2. **Custom classes not working:**
   - Verify configuration in [tailwind.css](mdc:libraries/design/src/tailwind.css)
   - Check for typos in class names
   - Ensure proper plugin installation

When you use this rule file, let me know this rule file is being used in the chat by mentioning the rule filename.

---
> Source: [cyberuni/pyrenees](https://github.com/cyberuni/pyrenees) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
