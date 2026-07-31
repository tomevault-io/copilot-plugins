## 02-frontend-development

> Svelte 5 best practices, UI principles, and interactions.


# 02-frontend-development

## Frontend Development (SvelteKit & UI)

STRICT REQUIREMENT:
- Svelte 5 Runes Adoption: Use `$state`, `$derived`, and `$effect` for reactivity. Avoid `$:` unless strictly necessary, and such instances must be clearly commented.
- UI Aesthetics: Allow for easy theme switching to change the overall look.
  - Default: a modern, clean interface with a preferred dark theme. 'Github Dark' / 'Github Light' inspired aesthetic. 
  - Custom Themes (json, import/export): allow extensive aesthetic adjustments.
    - Included Custom Themes: 
      - 'Gamer': a theme inspired by Battlefield 2042 & Escape from Tarkov UI design aesthetic. googleFont: 'Chakra Petch'.
      - 'Techi': a tech enthusiast inspired theme (sharper corners, innovative & pro FUI design aesthetic). googleFont: 'Orbitron'.
- Consistency: Use `"Inter"` font unless specified otherwise. Maintain consistent corner radii on all elements (although adjustable).
- Responsiveness: Fully adaptable UI without fixed pixel values — use `%`, `vw`, or Tailwind responsive prefixes (`sm:`, `md:`, `lg:`).
- Interactivity: Buttons and links must reliably respond to mouse clicks and touch taps.
- No Cumulative Layout Shifts (CLS): Prevent unexpected layout jumps.
- Dashboard Customization: 
  Support:
  - One-click view/edit mode.
    - View mode auto-hides all sidebars, topbar is visible upon hover.
  - Saving/loading custom dashboard layouts (i.e. 'dashboard snapshot').
    - Save widget placement(i.e. x, y coordinates), gauge type, sensor, size, etc.
    - Include custom UI elements (i.e. User-uploaded custom gauges, widgets, images, image-sequence-gauges)
  - Adding/removing/moving widgets dynamically.
  - Drag-n-drop.
  - Resizing widgets (move/resize handles, 'width(px), height(px)' values/inputs).
  - Snap-to-grid.
  - Show/hide grid.
  - Adjust grid size.
  - Custom background image upload via computer or url.
  Widget customization: Allow individual widget 'edit' parameters.
  - Edit/change gauge type.
  - Edit/change gauge attributes (i.e. Radial Gauge: Arc-start-position: 235 degrees; Arc-end-position: 115 degrees; Arc-stroke-thickness: 4px; Arc-dashes: true; etc.).
  - Edit/change sensor source.
  - Show/hide sensor display name.
  - Edit/change sensor display name.
  - Show/hide sensor metric/unit (i.e. ℃, MHz, MB, etc.)
  - Custom Gauge: User-uploaded image-sequence (similar to Aida64 sensor panel custom gauge)
    - Allow user to upload multiple images to make up a full animation sequence - used to render custom gauges.
    - Maximum of 24 images/sequence.
    - Allow easy repositioning of images in sequence.
    - Allow previewing of custom gauge animation.
      - Show a responsive live preview.

---
> Source: [fivelity/usmp](https://github.com/fivelity/usmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
