## retune

> An npm package that provides a visual overlay for dev mode. Users activate it, enter "edit mode", select DOM elements on their running app, tweak properties visually (like devtools but intuitive), and changes get sent to AI coding tools (Claude Code, Cursor, etc.) to persist as actual code changes.

# Project: Visual DevTools Overlay

## What This Is

An npm package that provides a visual overlay for dev mode. Users activate it, enter "edit mode", select DOM elements on their running app, tweak properties visually (like devtools but intuitive), and changes get sent to AI coding tools (Claude Code, Cursor, etc.) to persist as actual code changes.

**Mental model:** Visual devtools that persists changes to source code via AI agents.

**Target users:** Designers vibe-coding who struggle to get prototypes visually right, and developers fixing visual inconsistencies/bugs.

**References:**
- [Dialkit](https://joshpuckett.me/dialkit) — visual controls overlay for tweaking properties
- [Agentation](https://agentation.dev) — element targeting + structured annotations for AI agents
- Figma Make / V0 — point-to-edit design capabilities

This tool merges direct visual manipulation (Dialkit) with automatic AI agent integration (Agentation).

## Key Decisions

- **Distribution:** npm package (like Dialkit and Agentation)
- **Primary use case:** Dev mode — fixing visual inconsistencies and bugs
- **Live preview:** Changes show immediately in the browser (like devtools), then get sent to AI
- **Output:** Structured diff of before/after values + element identification context
- **AI integration:** MCP server (for Claude Code, Cursor) + copy-to-clipboard fallback

## Element Identification (Layered)

1. **DOM-level (always available):** CSS selector path, text content, applied classes, computed styles, bounding box
2. **React-specific (fiber tree):** Component name, props, component ancestry path
3. **Source-level (optional build plugin):** File path + line number via `__source` metadata (Babel/SWC plugin)

For MVP, layers 1+2 are sufficient — AI agents can grep their way to the right file with component name + selector + text content.

## Dynamic Controls (Based on Selected Element)

| Element type | Controls |
|---|---|
| Any element | padding, margin, border-radius, background, opacity, shadow |
| Text (p, h1, span) | font-size, weight, line-height, letter-spacing, color, alignment |
| Flex container | direction, gap, align, justify, wrap |
| Grid container | columns, rows, gap |
| Image | object-fit, aspect-ratio, border-radius |
| Fixed/absolute | position offsets, z-index |

Introspect computed styles, detect layout mode, generate appropriate control panel dynamically.

## Output Fidelity Levels

Since changes are exact (user made the visual tweak), fidelity levels control how much **context** is sent to help the model find the element in code:

| Level | Includes | Use case |
|---|---|---|
| Minimal | Selector + changed property diffs | Simple tweaks, small codebases |
| Standard | + React component tree + classes + text content | Most cases |
| Full | + computed styles + parent layout context + siblings + bounding box | Complex layouts, large codebases |

## Output Format Example

```
Element: <button> "Get Started"
Component: HeroSection > Button
Selector: main > section.hero > div > button.btn-primary
File hint: contains text "Get Started" in a Button component

Changes:
  padding: 12px 24px -> 16px 32px
  border-radius: 8px -> 12px
  background: #2563eb -> #1d4ed8
  font-size: 14px -> 16px
```

## Tech Stack

- Next.js, React, TypeScript, Tailwind CSS
- Distributed as the `retune` npm package

## Animations

Always use the `web-animation-design` skill when implementing any transition or animation. This ensures correct easing, timing, performance, and accessibility decisions.

## React Best Practices

Follow the principles in [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect):
- Don't use Effects to transform data for rendering — compute during render instead
- Don't use Effects to handle user events — handle in event handlers
- Derive state from props/state during rendering rather than syncing with Effects
- Use `useMemo` for expensive calculations, not Effects that set state
- When you must sync with external systems, keep Effects focused and clean

## Notes

- This project was previously called "Composer" (a standalone design tool extracted from the portfolio editor). It has been renamed to "Retune".
- The old Composer code can be fully replaced — nothing needs to be preserved.

---
> Source: [khadgi-sujan/retune](https://github.com/khadgi-sujan/retune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
