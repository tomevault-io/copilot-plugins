## motionly

> Motionly is a lightweight motion graphics engine inspired by Apple product videos.

# Motionly Agent Rules

## Mission

Build Motionly.

Motionly is a lightweight motion graphics engine inspired by Apple product videos.

The goal is simplicity, not feature count.

Every implementation decision must prioritize readability, smooth animation, performance, and maintainability.

---

# Core Philosophy

Motionly is NOT:

- Adobe After Effects
- Blender
- Unreal Engine
- Unity

Motionly IS:

- Lightweight
- Declarative
- Deterministic
- Fast
- File-based
- AI-friendly
- Browser-first

---

# Guiding Principles

## Simplicity First

If there are two possible implementations, always choose the simpler one.

Avoid abstraction unless it provides measurable value.

Avoid unnecessary configuration.

Avoid unnecessary options.

Avoid unnecessary nesting.

---

## The Source of Truth

Everything originates from `.motion`.

Never generate hidden state.

The renderer must only reflect what is written inside the scene file.

---

## Animation Philosophy

Animations should feel premium.

Inspired by:

- Apple
- Linear
- Arc Browser
- Framer
- WWDC presentations

Characteristics:

- smooth
- subtle
- spring-based
- soft easing
- minimal movement
- generous spacing
- no excessive motion

Avoid flashy effects.

---

## Performance

Target:

60 FPS minimum

120 FPS when supported

GPU accelerated

Live updates while editing

Fast parsing

Small memory footprint

---

## Rendering

Render vectors as vectors.

Do not rasterize SVGs during editing.

Only rasterize during export if required.

---

## Architecture

Prefer

Parser

↓

AST

↓

Scene Graph

↓

Animation Engine

↓

Renderer

Never mix responsibilities.

---

## Language

The `.motion` language must remain extremely small.

Every keyword should justify its existence.

Adding syntax requires strong justification.

---

## Writing `.motion` Files

The primary video entry point lives in:

`video-motion/codex-showcase.motion`

Use `video-motion/assets/` for scene-specific SVG/UI assets.

When authoring `.motion`:

- Treat the scene file as the complete source of truth.
- Write scenes like a product film, not a slideshow.
- Give each section one clear hero object and one clear viewer focus.
- Use semantic layers: `background`, `hero`, `supporting`, `details`, `text`, `effects`.
- Use camera movement intentionally: push in for importance, pull back for transitions, pan only when it supports the story.
- Use multiple scene atmospheres with `effect` layers instead of keeping one static background for the whole video.
- Keep SVG/UI panels inside the canvas after camera transforms; avoid layouts that only fit before zoom/pan.
- Prefer word-based text reveals for readable phrases. Use character reveals only for short, deliberate moments.
- Do not overuse blur. Prefer position, scale, camera, staging, and scene transitions.
- Avoid repeated logo entrances. Introduce the hero once, remove it when it no longer has purpose, and return it only for a final lockup.
- Use `sequence` for staggered code/text beats instead of hand-aligning many simultaneous delays.
- Use animation presets when they communicate intent: `keynoteText`, `heroLogo`, `productPanel`, `sceneExit`, `springIn`, `scaleReveal`, `rotateReveal`.

Recommended showcase structure:

1. Atmosphere: establish mood and question.
2. Hero: introduce the main object with camera push.
3. System: show scene file and preview as connected product objects.
4. Export: show rendering/output.
5. Final lockup: logo, Motionly, and one short line.

---

## UI

Minimal.

No floating windows.

No complex panels.

Focus on:

Explorer

Editor

Preview

Inspector

Export

---

## Dependencies

Only introduce dependencies when they significantly reduce complexity.

Avoid large frameworks.

---

## Code Quality

Prefer explicit code.

Avoid magic.

Avoid hidden behavior.

Avoid singleton-heavy architecture.

Prefer composition.

---

## Future Features

Do not implement future features early.

Ignore:

- AI
- Physics
- Audio
- Collaboration
- Cloud
- Plugins
- Particles
- 3D

Until Version 1 is complete.

---

## Success

Version 1 is successful if a user can:

Import SVGs

Reference them

Animate them

Preview instantly

Export MP4

Nothing else is required.

---
> Source: [COPPSARY/Motionly](https://github.com/COPPSARY/Motionly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
