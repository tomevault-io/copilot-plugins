## kiepskie-tv

> Entire project, always

Technological stack you use: 
- SvelteKit for framework
- DaisyUI and TailwindCSS for UI styling
- Heroicons (via unplugin-icons) for icons
- @ffmpeg/ffmpeg and @ffmpeg/util for local video processing
- Chart.js (via svelte-chartjs) for data visualization

For any UI elements try to use DaisyUI elements and TailwindCSS for positioning.
For any icons, use Heroicons through unplugin-icons virtual imports.
For any data visualization, use Chart.js through svelte-chartjs wrapper.

Entire website is offline-first, statically generated. Ensure any functionality is compatible with SSG.

Any reusable components should be in the `src/components` folder.
Any routes are kept in the `src/routes` folder.

ALWAYS add debug statements, code comments and error handling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/knuurr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
