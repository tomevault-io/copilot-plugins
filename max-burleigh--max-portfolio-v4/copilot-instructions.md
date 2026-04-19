## max-portfolio-v4

> **Framework:** NextJS 15 + Tailwind v4


## Portfolio Global Styles — Condensed Cheat Sheet

**Framework:** NextJS 15 + Tailwind v4

## Development Commands

- `npm run dev` - Start development server with hot reload
- `npm run build` - Create production build
- `npm run start` - Start production server
- `npm run lint` - Run ESLint checks

## Key Technologies

- **React 19** with TypeScript for type safety
- **Framer Motion v12** for animations and gestures
- **Platform detection** for iOS-specific optimizations
- **Throttled event handlers** for 60fps performance

## Performance Patterns

- Use motion values for hardware-accelerated animations instead of state
- Throttle mouse/scroll events to maintain smooth performance
- Lazy load iframes and images on user interaction

## Component Patterns

- Each component directory exports via `index.ts` barrel files
- Project cards use shared `ProjectCardProps` interface in `projects/shared/types.ts`
- Interactive states handled through refs and motion values
- **For project cards with phone mockups**: Always check screenshot dimensions first to determine if a custom aspect ratio variant is needed

---

### File Structure

* **app/globals.css**

  * Imports Tailwind base, components, utilities
  * Imports modular CSS for Aurora, nav, tech stack, project cards, iframe, etc.
  * Declares root CSS variables (colors, fonts)
* **app/layout.tsx**

  * Loads fonts (Manrope, Space Grotesk, Geist Sans/Mono)
  * Imports global & key component CSS
  * Sets up base HTML/body

---

### Fonts

* **Body:** Manrope (`--font-manrope`)
* **Headings:** Space Grotesk (`--font-space-grotesk`)
* **Supporting:** Geist Sans/Mono (for accents, code, system text)
* **All font vars set globally and used in CSS.**

---

### Layout/Core Structure

* **html/body:** 100% height, `overflow: hidden`, background transparent (for Aurora)
* **.portfolio-container:** Main scrollable area (`min-h: 100vh`, vertical scroll, smooth)
* **.section:** Max width 1000px, centered, `min-h: 100vh`, flex column, responsive padding
* **h1/h2:** Space Grotesk, bold, large; **a:** #00ffd5, 600 weight

---

### Color Scheme

* **Dark mode:**

  * `--background: #0a0a0a`
  * `--foreground: #ededed`
  * **Accent:** #00ffd5 (cyan/teal for links, nav)

---

### Special Effects

* **Aurora BG:**

  * `.aurora-bg` — full-viewport, fixed, animated gradient
  * `.aurora-blob` — framer-motion animated, blurred, mix-blend, responsive
  * **iOS:** AuroraBlobs not rendered, fallback bg color for perf
* **Cursor Light:**

  * `framer-motion` + radial gradient, follows mouse, desktop only
  * Hidden on mobile

---

### CSS/Tailwind Org

* **TailwindCSS** utility-first, config in `tailwind.config.js`
* **Custom styles:** Modular, imported into globals.css (not all dumped in one file)

---

**Summary:**

* Modern font stack, clean structure
* Modular CSS, Aurora/Cursor effects, smooth scrolling
* Dark theme with standout accent color
* Organized for clarity & scalability

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Max-Burleigh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
