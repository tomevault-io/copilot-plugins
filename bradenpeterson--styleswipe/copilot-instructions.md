## styleswipe

> Application UI and frontend rules.


# Logo

- Use **`assets/logo_cropped.png`** as the logo for all branding and UI applications across the entire application.

# Colors and Theming

- Define a single palette and use it consistently across screens.
- Prefer theme tokens (e.g. `colors.primary`, `colors.background`) over raw hex values.
- Support light/dark mode via theme if the app uses it.

# Font

- Use **Sans-Serif only** for all UI text.

# Touch Targets and Gestures

## Swipe card behavior

- The swiped image must **feel like a physical card**: it follows the user’s finger.
- **Do not reveal the next image** until the current card has been decided (swiped left or right).

## Pan gesture (top card)

- **Drag right** → card follows finger with rotation (e.g. +15° at full drag).
- **Drag left** → card follows with rotation (e.g. -15°).
- **Release:**
  - Velocity &gt; 400 to the right → trigger "like", animate card off to the right.
  - Velocity &lt; -400 to the left → trigger "skip", animate card off to the left.
  - Otherwise → spring back to center.

## Animation

- Exit duration: ~200ms.
- Snap-back: spring config (e.g. damping 15, stiffness 150).
- Next card: opacity/scale from 0.95 to 1 when it becomes the top card.

# Navigation Bar

- Visible and accessible on **every page**.
- Bottom of screen.
- Use it to navigate between: **Discover**, **Recommendations**, and **My Style**.

# Swipe UI

- Swipe image takes up the **full screen** except for the bottom navigation bar.
- No other chrome on the swipe screen; focus on the card.

# Recommendations Layout

- **Full screen** (below nav).
- **Top:** Filters/categories — user can select article type (pants, accessories, shirts, shorts, shoes, etc.) and/or colors.
- **Grid:** Three columns.
- **Pagination:** Load first 10 rows; when user scrolls to the bottom, load 10 more (infinite scroll).
- Provide an option to **reload/refresh** the list.

# Onboarding Flow

## Welcome

- No write on this screen.
- **Skip** → set `has_onboarded = true`, `preferred_* = []`, navigate to Discover.
- **Next** → navigate to Style Picker.

## Style Picker

- Multi-select from style tags (e.g. minimalist, streetwear, …).
- On **Complete**: write to `preferred_styles` (or pass to next screen and write at end).

## Color Picker

- Multi-select from color tags → `preferred_colors`.

## Category Picker

- Multi-select from category tags → `preferred_categories`.

## On Complete (all steps)

- Upsert profile: `has_onboarded = true`, `preferred_styles`, `preferred_colors`, `preferred_categories`.
- Optionally seed `tag_scores`: for each tag in `preferred_*`, set `tag_scores[tag] = 0.5` (or 0.3).
- **On Skip** (from any screen): `has_onboarded = true`, `preferred_* = []`, `tag_scores = {}`.
- Supabase: `supabase.from('profiles').upsert({ id: user.id, has_onboarded: true, preferred_styles: [...], ... }, { onConflict: 'id' })`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bradenpeterson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
