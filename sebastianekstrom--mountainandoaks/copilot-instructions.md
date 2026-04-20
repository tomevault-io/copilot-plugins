## mountainandoaks

> RPC Service boilerplate


# Code patterns

## Architecture

- This is a Next.js 14 (Pages Router) recipe and restaurant review website
- Uses TypeScript, React, and Tailwind CSS
- Static site generation (SSG) with `getStaticProps` and `getStaticPaths`
- All recipe and restaurant data is hardcoded in constants files (no external API or database)

## Code patterns

- Avoid using `for` loops - prefer array methods like `.map()`, `.filter()`, `.forEach()`
- Don't use the `any` type
- Never fix any ESLint styling issues (e.g indentation, trailing commas, etc.), but do fix everything else

## Styling

- Use Tailwind CSS classes
- Responsive design with mobile-first approach (use `lg:` prefix for desktop)
- Custom color classes like `bg-separator` are defined in the Tailwind config
- Always use the <Text /> component for text and make sure to use variants to adjust its styling, do not write any Tailwind classes that will modify the text styling
- Always reference the Tailwind config for available classes.

## Testing Preferences

- Mock as little as possible in TypeScript and React tests
- Use other test files as a reference for testing patterns
- Only test files in the `/src` directory

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sebastianekstrom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
