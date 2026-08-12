## figma-exceptions

> Intentional deviations from Figma-to-code parity. **Do not add exceptions without explicit discussion and agreement.**

# Figma-Code Exceptions

Intentional deviations from Figma-to-code parity. **Do not add exceptions without explicit discussion and agreement.**

## How to use

Before "fixing" a Figma/code mismatch, check if it's listed here. If listed, the deviation is intentional.

## Exceptions

### Form inputs: Transparent background

**Applies to**: Input, Textarea, TextareaGroup, Autocomplete, Combobox, Select

| Figma | Code |
|-------|------|
| `--surface-primary` fill | No background (transparent) |

**Reason**: Form inputs must blend with any surface color. Figma requires a fill for focus ring effects to render — this is a tooling limitation, not a design decision.

### Textarea: Auto-grow with 66px min-height

**Applies to**: Textarea

| Figma | Code |
|-------|------|
| Fixed height, `min-height: 64px` | `field-sizing: content`, `min-height: 66px` |

**Reason**: `field-sizing: content` auto-grows the textarea with content, keeping bottom padding always visible. 66px (vs Figma's 64px) aligns to the 20px line grid for a clean 2-line empty state.

### Textarea: Custom resize handle

**Applies to**: Textarea (resize grip)

| Figma | Code |
|-------|------|
| `IconTextareaDrag` at bottom-right | Custom SVG + pointer-event drag handle |

**Reason**: Figma's icon represents the native browser resize handle. Code uses a custom implementation with pointer events for pixel-perfect control over icon appearance (icon/tertiary), positioning (4px gap), and keyboard accessibility (role="separator" + arrow keys).

### InputGroup.Button: 3px border-radius

**Applies to**: InputGroup.Button

| Figma | Code |
|-------|------|
| 3px radius (intentional) | `border-radius: 3px` (raw value) |

**Reason**: Intentional non-token value per design. The inline button uses a radius between `--corner-radius-2xs` (2px) and `--corner-radius-xs` (4px) to optically balance with the container's `--corner-radius-sm` (6px).

### InputGroup: 5px edge padding for button/trigger

**Applies to**: InputGroup (Root) when a Button or SelectTrigger sits at an edge

| Figma | Code |
|-------|------|
| `--spacing-2xs` (6px) edge padding | `5px` (raw value) |

**Reason**: The container's 1px border reduces the available inner height from 36px to 34px, so vertical centering of the 24px button yields a 5px gap top and bottom. Edge padding is set to 5px to match, giving equal visual spacing on all sides.

### InputGroup: Transparent background

**Applies to**: InputGroup (Root)

Same as form inputs above — uses transparent background despite Figma showing `--surface-primary`.

---
> Source: [lightsparkdev/origin](https://github.com/lightsparkdev/origin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
