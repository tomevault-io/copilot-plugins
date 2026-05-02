## animation-principles

> This document helps Claude Code understand and use the Animation Principles Skill Marketplace.

# Claude Code Integration

This document helps Claude Code understand and use the Animation Principles Skill Marketplace.

## What This Repository Contains

This is a skill marketplace with 144 skills teaching Disney's 12 Principles of Animation across different contexts:

- **12 categories** organized by domain, role, tool, emotion, etc.
- **12 skills per category** including a universal fallback
- **Comprehensive coverage** from beginner to master level

## Skill Location

Skills are in two locations:
- `skills/` - Source SKILL.md files (canonical)
- `docs/` - Rendered markdown for browsing

## How to Use Skills

### Finding the Right Skill

1. **By what you're building**: Check `01-by-domain/` or `07-by-ui-element/`
2. **By your role**: Check `03-by-role-persona/`
3. **By your tool**: Check `09-by-tool-framework/`
4. **By desired emotion**: Check `06-by-emotional-outcome/`
5. **By problem to solve**: Check `12-by-problem-type/`

### Skill Naming Convention

Skills use the format: `animation-principles:<skill-name>`

Examples:
- `animation-principles:web-motion-design`
- `animation-principles:joy-delight`
- `animation-principles:framer-motion`

### Universal Fallbacks

Each category has a universal fallback for general use:
- `universal-fallback` (domain)
- `universal-mindset` (thinking style)
- `universal-practitioner` (role)
- `universal-reference` (skill level)
- `universal-patterns` (animation type)
- `universal-emotion` (emotional outcome)
- `universal-elements` (UI element)
- `universal-industry` (industry)
- `universal-tool` (tool/framework)
- `universal-timing` (time scale)
- `universal-solutions` (problem type)

## The 12 Principles Quick Reference

When applying animation principles:

| Principle | Key Application |
|-----------|-----------------|
| Squash & Stretch | Show impact and flexibility |
| Anticipation | Prepare before action |
| Staging | Direct attention clearly |
| Straight Ahead / Pose to Pose | Choose animation approach |
| Follow Through / Overlapping | Stagger and continue motion |
| Slow In / Slow Out | Always use easing, never linear |
| Arc | Curved paths feel natural |
| Secondary Action | Add supporting movement |
| Timing | Duration = weight and emotion |
| Exaggeration | Push for clarity |
| Solid Drawing | Maintain volume |
| Appeal | Create charisma |

## Common Tasks

### Adding Animation to UI
1. Use `web-motion-design` or `css-native` for implementation
2. Use `micro-interactions` for small moments
3. Check `06-by-emotional-outcome/` for the right feel

### Debugging Animation
1. Start with `implementation-debugging`
2. Check `performance-optimization` for jank
3. Use `timing-calibration` for feel issues

### Learning/Teaching
1. `absolute-beginner` for first introduction
2. `teaching-others` for instruction
3. `11-by-principle-focus/` for deep dives

## Testing Commands

This is a documentation-only repository. No build or test commands required.

## File Naming

- Skill directories: lowercase with hyphens (`joy-delight/`)
- Skill files: `SKILL.md` (uppercase)
- Doc files: `<skill-name>.md` (lowercase)

---
> Source: [dylantarre/animation-principles](https://github.com/dylantarre/animation-principles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
