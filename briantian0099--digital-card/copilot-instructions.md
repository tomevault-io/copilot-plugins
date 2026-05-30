## digital-card

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

An interactive digital business card with holographic effects. Features include a spring physics-driven 3D card, pointer/deviceorientation tilt, and premium UI design (Apple/Arc-inspired). Desktop uses mouse tracking; mobile uses DeviceOrientation API with iOS permission handling.

## Running the Project

No build process. Open `main.html` in a browser. Changes to `script.js` and `style.css` reflect on refresh.

## Customizing Content

All personal information is centralized in `script.js:1-13` within the `profileData` object:
```javascript
const profileData = {
  name: 'Your Name',
  role: 'Your Role',
  tagline: 'Your tagline\n(supports line breaks)',
  avatarInitials: 'YN',
  email: 'you@example.com',
  location: 'City, Country',
  skills: ['Skill1', 'Skill2', 'Skill3'],
  socials: [
    { label: 'GH', url: 'https://...', title: 'GitHub' },
    { label: 'LI', url: 'https://...', title: 'LinkedIn' },
  ]
};
```
Modify this object; `renderCard()` injects content dynamically. No HTML changes needed for content updates.

## Architecture

### Spring Physics System
Universal across all interactions (desktop/mobile):
- **Three Springs**: `rotationSpring` (tilt), `backgroundSpring` (shine position), `pointerSpring` (glare + effect intensity)
- **Dual Settings**: `springInteractSettings` (stiff, responsive while hovering) vs `springReturnSettings` (soft, smooth return-to-rest)
- **Animation Loop** (`animateCard()`, ~line 200): `requestAnimationFrame` updates springs and halts when all settle below `STOP_THRESHOLD`
- **Semi-implicit Euler**: Spring integration via `updateSpring()` with frame-rate compensation

### Input Modes

**Desktop (Pointer):**
- `handlePointerMove()`: Tracks mouse, updates all three spring targets
- `handlePointerLeave()`: 500ms delay before returning springs to rest (prevents jitter)

**Mobile (DeviceOrientation):**
- `initMobileOrientation()`: iOS 13+ shows permission overlay; Android starts immediately
- `attachOrientationListener()`: Maps `gamma`/`beta` to spring targets, ~30fps throttle for battery
- Constant `effectIntensity: 0.65` while tilting (no mouse-like full opacity swing)

### CSS Contract
The three springs set 8 CSS variables on `.card__rotator`:
- `--tilt-left-right`, `--tilt-up-down`: Rotation degrees (applied to `transform: rotateY/rotateX`)
- `--background-x`, `--background-y`: Shine gradient position (0–100%)
- `--pointer-x`, `--pointer-y`, `--effect-intensity`, `--pointer-from-center`: Glare parameters

### Holographic Effect (Visual)
- **`.card__shine`**: Repeating rainbow gradient (`repeating-linear-gradient`, 135deg) positioned by `--background-x/y`, blended with `mix-blend-mode: color-dodge`; opacity `0.05 + (effectIntensity × 0.55)` → near-invisible at rest, vivid on interaction
- **`.card__glare`**: Radial white spotlight at `--pointer-x/y`, `mix-blend-mode: overlay`, opacity driven by `--effect-intensity`

## Design System

**Colors** (dark theme, Apple-inspired):
- Background: `#080808`
- Card: `#111113`
- Text primary: `#f2f2f7`
- Text secondary: `rgba(242,242,247,0.55)`
- Accent: `#6e6ef5` (subtle purple, Arc-like)
- Borders/surfaces: `rgba(255,255,255, 0.06–0.12)`

**Card Layout** (360×520 portrait):
Header (avatar + name/role) → divider → tagline → skills → footer (contact + socials)

**Responsive**: Scales card width to `88vw` below 480px viewport

## Making Changes

- **Tilt Intensity**: Adjust divisors at `script.js:~128-133` (currently `/3.5` for desktop, `×14` for mobile)
- **Spring Feel**: Edit `springInteractSettings` (`stiffness: 0.066, damping: 0.25`) and `springReturnSettings` (`stiffness: 0.01, damping: 0.06`) at `script.js:34-39`
- **Holographic Gradient**: Modify `repeating-linear-gradient` in `style.css` `.card__shine` rule (hue stops every 8%)
- **Colors**: Use `:root` CSS variables in `style.css` (`--color-accent`, `--color-card`, etc.)
- **Reset Delay**: Change `handlePointerLeave(event, delay = 500)` at `script.js:~238`
- **Mobile Throttle**: Edit `if (now - lastOrientationTime < 33)` at `script.js:~107` (33ms ≈ 30fps)

---
> Source: [BrianTian0099/digital-card](https://github.com/BrianTian0099/digital-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
