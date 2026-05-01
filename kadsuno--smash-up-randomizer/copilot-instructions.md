## design-system

> UI consistency — buttons, spacing, colors; canonical landing reference and sur-* components.


## Design system (Smash Up Randomizer)

- **Canonical public UI reference:** [`resources/views/start/home.blade.php`](resources/views/start/home.blade.php) — match hero CTAs, glow tier, chips, and shuffle dialog patterns unless the ticket says otherwise.
- **Secondary reference (logged-in):** account shell patterns in [`resources/views/account/`](resources/views/account/) — glass panels, tiles, radial/grid backgrounds.
- **Full specification:** [`docs/design-system.md`](docs/design-system.md) — **foundations** (typography/Nunito + Font Awesome, color, spacing, radii, motion + reduced-motion, z-index, scrollbars), **complete `sur-*` register**, shuffle/cookie dialogs, Blade zones, UI concept catalog, border tokens, extractions.
- **Named styles:** [`resources/css/app.css`](resources/css/app.css) `@layer components` — prefer **`sur-btn-primary`**, **`sur-btn-secondary`**, **`sur-btn-ghost`** over raw `bg-indigo-*` + `rounded-xl` button stacks.
- **Hero / spotlight CTAs:** may add **one** extra indigo shadow utility on primary (e.g. `shadow-lg shadow-indigo-500/30`) **as on the landing**; do not invent new glow tokens per view.
- **Layout rhythm:** [`x-sur.section`](resources/views/components/sur/section.blade.php), [`x-sur.container`](resources/views/components/sur/container.blade.php); button groups typically **`gap-3`**.
- **Exceptions:** new special controls get a **`sur-*` class** in `app.css` + a row in the **UI concepts** table in [`docs/design-system.md`](docs/design-system.md).
- **i18n:** user-visible strings **EN + DE** under `resources/lang/`.

Full checklist and tokens: **`docs/design-system.md`**.

---
> Source: [Kadsuno/smash-up-randomizer](https://github.com/Kadsuno/smash-up-randomizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
