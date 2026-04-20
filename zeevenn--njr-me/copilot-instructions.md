## njr-me

> You are working on a modern, elegant, content-focused **static blog site**.

You are working on a modern, elegant, content-focused **static blog site**.

This is a single-repo project using **Next.js** to build a Markdown-powered blog with MDX, customizable components, theme switching, and fully responsive styles.

## Tech Stack Overview

- Framework: **Next.js** (using the Pages or App Router)
- Styling: **Tailwind CSS** with `@tailwind/typography`
- Markdown Rendering: **MDX** via `@next/mdx`
- Metadata Parsing: `gray-matter` for frontmatter support
- UI Components: **Radix UI** for unstyled, accessible primitives (Dialog, Tooltip, Tabs, etc.)
- Theming: **next-themes** for system-aware light/dark mode
- Icons: **lucide-react**
- Optional Features:
  - Syntax Highlighting: `rehype-highlight` or `shiki`
  - Table of Contents: `rehype-slug`, `rehype-autolink-headings`, `rehype-toc`
  - RSS Feed: `rss` npm package
  - Deployments: **Vercel** or **Netlify**

## Content Architecture

- Blog content is written in `.mdx` format and stored under `content/blog`
- Each post includes frontmatter metadata (`title`, `date`, `tags`, etc.)
- MDX components are enhanced with custom React components (e.g. `<Callout />`, `<CodeBlock />`)
- Tailwind's `prose` class is used for beautiful typography out-of-the-box

## Component Guidelines

- Use `Radix UI` directly for new patterns
- Keep components small, composable, and theme-aware
- Follow Tailwind conventions (`dark:` variants, utility-first CSS)

Before starting, understand how the blog rendering works via MDX + custom components, and how the styling and theming are handled through Tailwind and `next-themes`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zeevenn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
