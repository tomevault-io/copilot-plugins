## beyond-excel-biology

> - Install Hugo: Follow https://gohugo.io/installation/

# Agent Guidelines for beyond-excel-biology

## Build/Test Commands
- Install Hugo: Follow https://gohugo.io/installation/
- Serve development: `hugo server -D`
- Build for production: `hugo --gc --minify`
- Test build: `hugo --buildDrafts`
- No test suite - Hugo builds serve as validation

## Theme & Design
- **Current Theme**: Nord Theme with Aurora color enhancements
- **Color Palette**: 16 Nord colors (nord0-15) organized as:
  - Polar Night (nord0-3): Dark backgrounds, Snow Storm (nord4-6): Light text
  - Frost (nord7-10): Blue accents, Aurora (nord11-15): Colorful highlights
- **CSS Variables**: All colors defined as CSS custom properties in `static/style.css`
- **Both themes supported**: Dark mode (default) and light mode via `[data-theme="light"]`

## Code Style & Structure
- Hugo static site generator with Go templating, HTML, CSS, Markdown
- **Config**: `hugo.yaml` (YAML format), **Layouts**: `layouts/`, **Content**: `content/`
- **Main stylesheet**: `static/style.css` using CSS custom properties
- Follow kebab-case for files, CSS variables use `--kebab-case`
- Use semantic HTML5 elements, Hugo templating follows Go template syntax
- Front matter required (YAML), 2-space indentation for YAML/CSS
- Posts in `content/posts/` with descriptive filenames
- Partial templates in `layouts/partials/`

## Error Handling
- Hugo fails builds on template/front matter syntax errors
- Validate YAML front matter and Go template syntax
- Test locally with `hugo server -D` before committing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simeross)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/simeross)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
