## vcp-code-2-0

> We are a fork of Roo. We regularly merge in the Roo codebase. To enable us to merge more easily, we mark all

# Novacode Change Marking Guidelines

We are a fork of Roo. We regularly merge in the Roo codebase. To enable us to merge more easily, we mark all
our own changes with `novacode_change` comments.

## Basic Usage

### Single Line Changes

For single line changes, add the comment at the end of the line:

```typescript
let i = 2 // novacode_change
```

### Multi-line Changes

For multiple consecutive lines, wrap them with start/end comments:

```typescript
// novacode_change start
let i = 2
let j = 3
// novacode_change end
```

## Language-Specific Examples

### HTML/JSX/TSX

```html
{/* novacode_change start */}
<CustomNovaComponent />
{/* novacode_change end */}
```

### CSS/SCSS

```css
/* novacode_change */
.novacode-specific-class {
	color: blue;
}

/* novacode_change start */
.another-class {
	background: red;
}
/* novacode_change end */
```

## Special Cases

### New Files

If you're creating a completely new file that doesn't exist in Roo, add this comment at the top:

```
// novacode_change - new file
```

### Novacode specific file - these rules take precedence over all other rules above

- if the filename or directory name contains novacode no marking with comments is required
- all the following folders are novacode-specific and need no marking with comments:
    - jetbrains/
    - cli/
    - src/services/autocomplete/
    - src/services/continuedev/

---
> Source: [DerstedtCasper/vcp-code-2.0](https://github.com/DerstedtCasper/vcp-code-2.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
