## saeed-vayghan-github-io

> Hello fellow AI Agent! Welcome to the `saeed-vayghan.github.io` project.

# AI Agent Onboarding Guide

Hello fellow AI Agent! Welcome to the `saeed-vayghan.github.io` project. 

This document serves as your primary context when making modifications, creating new content, or debugging issues in this repository. Please read this carefully before executing commands or modifying code.

## 1. Project Philosophy & Architecture

- **100% Pure Static**: This repository uses absolutely **no build system**. Do not try to initialize npm, install Vite, Webpack, or any other bundler. Do not create a `package.json`.
- **Direct HTML/CSS/JS**: Everything runs directly in the browser. 
- **File System Routing**: URLs correspond directly to the folder structure. E.g., `projects/sweden-salary-calc.html` is accessible at `/projects/sweden-salary-calc.html`.

## 2. Directory Structure & Source of Truth

- The ultimate source of truth for the folder structure is **`index.md`**.
- **Whenever you create a new file, move a file, or delete a file, you MUST update the tree diagram in `index.md` to reflect the new state.**

## 3. Creating New Content

When asked to create a new project or a new blog post:
1. **Use the Templates**: Do not write HTML from scratch. Copy `projects/project-template.html` or `blog/post-template.html` and rename it appropriately.
2. **Update the Index**: After creating your new file, you must update the list in `projects/index.html` or `blog/index.html` to link to it.
3. **Update `index.md`**: Add the new file to the directory tree in `index.md`.
4. **Domain Tags & Badges**: Every post must include category/domain pill tags:
   - On the list index (`blog/index.html`): Wrapped in a `<div class="tag-container">` below the post description, e.g., `<span class="tag">LLMs</span>`.
   - On the individual post page: Wrapped in a `<div class="tag-row" style="margin-top: 1rem;">` inside the `.post-header`, using the classes `.pill-tag` along with a color theme class:
     - `.pill-fintech` (blue tint)
     - `.pill-realestate` (pink/red tint)
     - `.pill-media` (orange tint)
5. **Table of Contents (TOC)**: For long or multi-section guides, add a Table of Contents right before the main body paragraphs.
   - Use the `<div class="toc-container">` wrapper.
   - Include a semantic `<ol>` (ordered list) with links mapping 1-to-1 to your headings.
6. **Date Consistency**: Publication dates must be kept perfectly synchronized across:
   - The `<time>` tag in `blog/index.html`.
   - The `<time class="post-date">` tag inside the post file's `.post-header`.
   - The parenthetical date annotation in `index.md` (e.g., `(Claude Code Best Practices guide - April 20, 2026)`).
7. **RSS Feed & Sitemap**: After creating or modifying a blog post or project page (including updating the lists in `blog/index.html` or `projects/index.html`), you must update both the RSS feed and the sitemap by running `python3 generate_rss.py`. This script dynamically builds `rss.xml` and `sitemap.xml` based on the content indexes.
8. **SEO & Social Metadata**: Every new blog post or project page must include proper SEO metadata in the `<head>` block. Use the layout present in `blog/post-template.html` or `projects/project-template.html` and populate:
   - `<meta name="description" content="...">`: A concise summary of the post.
   - `<meta name="keywords" content="...">`: Main keywords related to the post context.
   - Open Graph (OG) tags (`og:title`, `og:description`, `og:url`, `og:type`) to render rich previews on LinkedIn/Facebook.
   - Twitter Card tags (`twitter:card`, `twitter:url`, `twitter:title`, `twitter:description`) to optimize views on Twitter.

## 4. Styling & CSS Rules

- **Global Styles**: Global styles are defined in `statics/styles.css`.
- **Layout Container**: The main wrapper class for layout is `.site-container`. **Do not use `.container`** because some standalone project pages import Tailwind CSS via CDN, and Tailwind's `.container` class will conflict with our global layout.
- **Tailwind CSS**: Some pages (like `sweden-salary-calc.html` or `01-LLM-tools-analysis.html`) pull in Tailwind via a CDN script tag. This is allowed for complex, standalone tools. However, for standard blog posts or simple pages, rely entirely on `statics/styles.css`.

## 5. Theme System (Light/Dark Mode)

- Dark mode is fully supported.
- Colors are defined via CSS variables in the `:root` and `[data-theme="dark"]` selectors inside `statics/styles.css`.
- The theme toggle logic lives in `statics/theme.js`. It sets the `data-theme` attribute on the `<html>` element and persists the choice in `localStorage`.
- When adding new custom CSS classes, always ensure you use the CSS variables (e.g., `var(--bg-color)`, `var(--text-color)`) so that dark mode is automatically supported.

## 6. Global Navigation (Header & Footer)

- All pages must have the exact same `<header>` and `<footer>` structure.
- The templates (`project-template.html` and `post-template.html`) already include this.
- If you edit the navigation (e.g., adding a new top-level page like "Contact"), you must manually update the `<header>` block in **every single `.html` file** across the project. There is no templating engine.

## 7. Code Snippets & Highlighting

When adding code snippets to blog posts or project pages, follow this exact structure to maintain aesthetic consistency:
1. **Syntax Highlighting**: Use [Prism.js](https://prismjs.com/) for syntax highlighting via CDN. Add the CSS to `<head>` (`prism-tomorrow.min.css`) and the necessary scripts before `</body>`.
2. **Mac-Window Styling**: Wrap all code snippets in a `.mac-window` div to give them a native IDE look (the CSS for this is in `statics/styles.css`).
3. **HTML Structure**:
   ```html
   <div class="mac-window">
       <div class="mac-header">
           <span class="mac-dot red"></span>
           <span class="mac-dot yellow"></span>
           <span class="mac-dot green"></span>
       </div>
       <pre><code class="language-python">Your code here</code></pre>
   </div>
   ```
4. **No Custom Colors**: Do NOT apply color overrides like `pre code { color: var(--text-color); }` in the document head, as it breaks the dark mode syntax highlighting from Prism.
5. **Inline Code & File References**: Do NOT use custom CSS classes or inline styling for simple path names, command-line arguments, or inline code blocks (e.g., `~/.claude/CLAUDE.md`). Just wrap them in a standard `<code>` tag. The global styles inside `statics/styles.css` will automatically style them with the correct font-family, font-weight, padding, borders, and theme-aware colors.

## 8. CSS Reusability (Strict No-Inline-Style Rule)

When modifying layout or typography, **DO NOT** use inline `<style>` tags or excessive inline `style="..."` attributes. Instead, you must use the standard global classes provided in `statics/styles.css`.

- **Typography**: Wrap content in `<article class="blog-content">` or `<article class="project-content">` to automatically inherit typography rules for `h1`, `p`, `ul`, `pre`, etc.
- **Back Links**: Use `<a href="..." class="back-link">` for "Back to Blog" or "Back to Projects" links.
- **Headers**: Use `<div class="post-header">` for the article header wrapper.
- **Titles and Dates**: Use `<h1 class="post-title">` and `<time class="post-date">` inside the post header.
- **Description Colors**: For secondary description text on the index lists, use the specific variable `var(--text-description)` (e.g., `.post-item p`) to ensure optimum contrast and premium appearance.

Good luck!

---
> Source: [saeed-vayghan/saeed-vayghan.github.io](https://github.com/saeed-vayghan/saeed-vayghan.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
