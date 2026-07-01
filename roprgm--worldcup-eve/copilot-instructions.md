## worldcup-eve

> A minimal example agent built with [eve](https://github.com/vercel/eve). See `README.md` for

# Contributor notes

A minimal example agent built with [eve](https://github.com/vercel/eve). See `README.md` for
what each file demonstrates. The full eve docs are bundled at `node_modules/eve/docs/` — read
the relevant guide before changing agent code.

## Component organization

- `components/ai-elements/` — vendored primitives from the
  [AI SDK Elements](https://vercel.com/academy/ai-sdk/ai-elements) library. Treat these as
  upstream: only put components that come from there here.
- `components/ui/` — our own generic, shadcn-like primitives with no domain knowledge (e.g.
  `Notice`, `ActivityStatus`).
- `components/` — higher level, app-specific components (e.g. `chat.tsx`, `header.tsx`).
- `components/<domain>/` — group components under a domain folder when one is important enough to
  warrant it.
- Within a file, order top-down by reading flow: constants and small helpers first, then the
  smaller building-block components, with the bigger / exported component(s) last. A reader meets a
  piece before the component that composes it. Extract sub-components freely — even single-use ones —
  to keep each one small and avoid deeply nested JSX conditionals.

## Package manager

- Use Bun. Install with `bun install` and run scripts with `bun run <script>`. Never use npm,
  pnpm, or yarn.

## Code style

- Keep code, comments, and docs in English, and prefer the simplest useful implementation.
- Don't over-comment. When names are clear and a function is small with a single responsibility,
  the code is its own explanation — add a comment only to capture the non-obvious _why_ (a subtle
  invariant, a workaround, an intentional edge case), never to restate _what_ the code does.
- Keep comments short — a line or two. If a comment needs a paragraph, the naming or structure
  probably needs the work instead.
- Import local modules through the `@/*` path alias.
- Merge `className` values with `cn` from [`cnfast`](https://github.com/aidenybai/cnfast) (a fast
  drop-in for `clsx` + `tailwind-merge`). Use it for any conditional or combined class names instead
  of template literals or ternaries — e.g. `cn("dark", isActive && "px-4")`.

---
> Source: [roprgm/worldcup-eve](https://github.com/roprgm/worldcup-eve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
