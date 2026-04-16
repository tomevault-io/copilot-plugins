## element-bits

> This document outlines the coding standards and guidelines for contributing to the Element Bits plugin.


---
trigger: always_on
---

# Contributing to Element Bits

This document outlines the coding standards and guidelines for contributing to the Element Bits plugin.

## Project Structure

```
├── assets
│   ├── css
│   │   ├── admin.css
│   │   ├── element-bits.css
│   ├── fonts
│   ├── images
│   └── vendor
├── dynamic-tags
├── element-bits.php
├── inc
├── index.php
└── widgets
```

## Project Directory
`wp-contnet/plugins/element-bits`

Each widget should live in its own folder with the following structure:
- `widget.css` - Widget-specific styles. Optional, not all widget need this.
- `widget.js` - Widget-specific scripts. Optional, not all widget need this.
- `widget.php` - Widget class implementation

## Coding Standards

### PHP
- Use PHP 7.4+ features
- Follow WordPress coding standards
- Use `elebits_` prefix for all plugin functions
- Add proper PHPDoc blocks to all functions and classes

### JavaScript/CSS
- Use clean, readable code
- Comment your code appropriately
- Follow WordPress coding standards

### Documentation
- WordPress and Elementor, use context7 MCP

## Git Workflow

**Commit Message Prefixes:**
- "fix:" for bug fixes
- "feat:" for new features
- "perf:" for performance improvements
- "docs:" for documentation changes
- "style:" for formatting changes
- "ref:" for code refactoring
- "test:" for adding missing tests
- "chore:" for maintenance tasks

**Rules:**
- Use lowercase for commit messages
- Keep the summary line concise
- Include description for non-obvious changes
- Reference issue numbers when applicable

## Widget Development

When creating a new widget:
1. Create a new directory in the `widgets` folder
2. Create the required files (module.php, widget.php, widget.css, widget.js)
3. Follow the existing widget structure as a template
4. Register the widget in the main plugin file

## Testing

Test all widgets thoroughly in different environments before submitting.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Novel-Digital-Agency) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
