## slidev-decks-simplon

> Auto-generated from all feature plans. Last updated: 2025-11-24

# slidev-simplon-sql-1st-day Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-11-24

## Active Technologies

- JavaScript/TypeScript with Node.js 18+ (Slidev requirement) + Slidev (latest stable), Vue 3, Vite, markdown-i (001-multi-deck-index)

### Slidev-Specific Technologies

- **Framework**: Slidev (Vue 3-based presentation framework)
- **Markdown**: Extended markdown with Vue components
- **Styling**: UnoCSS/Windi CSS for utility-first styling
- **Interactivity**: Vue components for interactive SQL demos

## Project Structure

```text
src/
tests/
```

### Typical Slidev Structure

```text
slides/
├── slides.md           # Main slide content
├── components/         # Vue components for interactive demos
├── public/            # Static assets (images, data files)
├── setup/             # Slidev setup files
└── styles/            # Custom styles

examples/
└── sql/               # Executable SQL examples with test data
```

## Commands

bun test && bun run lint

### Slidev Commands

```bash
# Development server with hot reload
bun run dev

# Build for production
bun run build

# Export to PDF
bun run export

# Export to SPA (static hosting)
bun run build -- --base /[repo-name]/
```

## Code Style

JavaScript/TypeScript with Node.js 18+ (Slidev requirement): Follow standard conventions

### Markdown Content Style

- Use `#` for slide titles (one per slide)
- Use `---` to separate slides
- Use `::code-group` for multiple code examples
- Wrap SQL in triple backticks with `sql` language tag
- Use Vue components with `<ComponentName />` syntax

### SQL Example Style

- Include setup scripts (CREATE TABLE, INSERT)
- Show expected output as comments
- Keep queries under 20 lines for readability
- Test all queries before committing

### Accessibility Guidelines

- Alt text for all images
- Descriptive link text (not "click here")
- Sufficient color contrast (use devtools to verify)
- Avoid relying solely on color to convey information

## Recent Changes

- 001-multi-deck-index: Added JavaScript/TypeScript with Node.js 18+ (Slidev requirement) + Slidev (latest stable), Vue 3, Vite, markdown-i

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxime-lenne) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
