## utc-website

> This skill covers:

# Agent Instructions — UTC Website 2025

This file is read by AI coding agents (Claude Code, Cursor, GitHub Copilot, etc.)
at the start of every session. It describes the conventions for this codebase.
**All agents should follow these rules regardless of what session they're in.**

---

## Storybook Stories — Two Distinct Categories

There are two fundamentally different kinds of stories in this codebase.
**Do not confuse them.**

---

### 1 · Primitive reference stories

```
components/Cube/Faces/primitives/
  Primitives.stories.tsx
```

These are **living documentation** for the actual primitive components.
They import and render the real `Cell`, `TextBlock`, `ImageBlock`, etc. directly.
If a primitive's API changes, these stories are expected to break — that is the point.
They are the source of truth for how primitives work and should be kept up to date
whenever a primitive is modified.

**Do not put explorations or layout experiments here.**

---

### 2 · Experiment stories

```
components/Cube/Faces/experiments/
  CubeFaceXRExperiments.stories.tsx         ← formerly CubeFaceExperiments
  CubeFaceWorkExperiments.stories.tsx
  CubeFaceWorkAcidFlatExperiments.stories.tsx   ← split from Work (file size)
  CubeFaceCollaboratorsExperiments.stories.tsx
  CubeFaceNewsExperiments.stories.tsx
```

When a section's experiment file gets too large, split by layout family:
`CubeFace{Section}{Subcategory}Experiments.stories.tsx` (e.g. Acid Flat).
Storybook title nests under the section: `Experiments/Cube Face Work/Acid Flat`.

These are **sandboxed layout explorations** — visual ideas for how a cube face
might look. They use the primitives, but their purpose is creative exploration,
not documentation. They should never import internal component logic directly
(no reaching into component source to access private helpers, etc.).

The key property of experiments is that they are **loosely coupled** — they
should not break when internal component implementation details change.

### Story naming

Every story **must** have a numbered `name` in the format `'N – Short Title'`:

```ts
export const MyStory: Story = {
  name: '7 – Portraits + Icon Quad',
  ...
};
```

Numbers make it easy to reference stories by number in conversation
("can you update story 7") and keep Storybook's sidebar sorted.

### Story identifier comments

Every story export must be preceded by a single-line identifier comment:

```ts
// ─── 7 · Portraits + Icon Quad ──────────────────────────────────────────────
export const PortraitsIconQuad: Story = {
```

This makes stories easy to find with `Cmd+F`, `grep`, or the editor outline.
Use `─` (U+2500) and `·` (U+00B7) to match the existing style.

### Layout model — use Cell + primitives, not absolute divs

All Cube face stories must use the `<Cell>` + primitives pattern.
Do **not** wrap FaceGrid children in raw `absolute inset-0` divs or bare grid wrappers —
this breaks `overflow-hidden` clipping and `cqi` unit resolution.

```tsx
// ✅ correct
<FaceGrid>
  <Cell col={1} row={1} colSpan={6} rowSpan={3}>
    <ImageBlock src="..." alt="..." mask="fade-down" />
  </Cell>
  <Cell col={1} row={5} colSpan={3} zIndex={1}>
    <TextBlock fontSize={22}>XR</TextBlock>
  </Cell>
  <Cell col={1} row={2} colSpan={6} zIndex={5}><StripeBars /></Cell>
</FaceGrid>

// ❌ wrong — breaks clipping and cqi units
<FaceGrid>
  <div className="absolute inset-0 grid grid-cols-6 grid-rows-6">
    ...
  </div>
</FaceGrid>
```

See `primitives/Primitives.stories.tsx` (title: "Cube Face Primitives")
for working examples of every primitive.

### Available primitives

| Import | Purpose |
|--------|---------|
| `Cell` | Grid placement — `col`, `row`, `colSpan`, `rowSpan`, `zIndex` |
| `GridLines` | Decorative 6×6 border overlay |
| `ColorBlock` | Solid colour fill |
| `GradientBlock` | Linear gradient — takes `stops` array, not `fromColor`/`toColor` |
| `ImageBlock` | Image with optional `mask` fade direction, `mixBlendMode` for layering |
| `TextBlock` | CQI-scaled text — uses `fontSize` (not `size`), children not `text` prop |
| `IconQuad` | 4 icons — `icons={{ tl, tr, bl, br }}` object, not an array |
| `IconSingle` | Single centred icon — `name`, `color`, `weight`, `iconSize` |
| `StripeBars` | Thin horizontal colour bar — fills its `Cell`, place with `<Cell>` like any content primitive |

### Icon names

Only use icons registered in `components/Icon/Icon.tsx`. Key ones:
`users`, `hand-waving`, `chat`, `share`, `globe`, `cube`, `atom`,
`lightning`, `sparkle`, `rocket`, `brain`, `broadcast`, `blueprint`,
`hard-hat`, `crane`, `virtual-reality`, `google-cardboard`, `cube-focus`.

Do **not** invent icon names like `users-three`, `handshake`, `network` —
they will cause runtime errors.

### Storybook section titles

| File | Storybook title |
|------|----------------|
| `CubeFaceExperiments.stories.tsx` | `Experiments/Cube Face XR` |
| `CubeFaceWorkExperiments.stories.tsx` | `Experiments/Cube Face Work` |
| `CubeFaceWorkAcidFlatExperiments.stories.tsx` | `Experiments/Cube Face Work/Acid Flat` |
| `CubeFaceCollaboratorsExperiments.stories.tsx` | `Experiments/Cube Face Collaborators` |
| `CubeFaceNewsExperiments.stories.tsx` | `Experiments/Cube Face News` |
| `primitives/Primitives.stories.tsx` | `Cube/Face Primitives` |

---

## Cube face baking — static image workflow

Cube faces are **pre-rendered to static JPEG images** and committed to the repo (`public/faces/*.jpg`). This avoids GPU compositor black-box artifacts caused by `mix-blend-mode` + `transform-style: preserve-3d` + continuous animation.

When the user asks to "bake", "capture", "snapshot", or "freeze" a face, read and follow:

```
.cursor/skills/bake-cube-face/SKILL.md
```

This skill covers:
- Running `scripts/capture-face.ts` to screenshot a face from Storybook
- Updating the face component to serve the baked image
- The favicon capture workflow (`scripts/capture-favicon.ts`)
- Re-baking after a design change

---

## Storybook + OpenNext/Cloudflare — `workerd` hang

Never add unconditional side-effects to `next.config.ts` that start long-running
processes. `@storybook/nextjs` evaluates the config at build time — anything that
spawns a child process (e.g. `initOpenNextCloudflareForDev()` → `workerd`) will
keep Node alive and the build will hang. Always gate behind `STORYBOOK_BUILD` /
`STORYBOOK` env vars. Full writeup: [`docs/storybook-workerd-hang.md`](docs/storybook-workerd-hang.md).

---

## General conventions

- **TypeScript** throughout — no `any`, no implicit types on component props.
- **Tailwind** for styling — use theme tokens (`bg-theme-cyan`, `text-theme-white`, etc.), not raw hex values.
- **CQI units** for all sizing inside `FaceGrid` — `1cqi` = 1% of face width. Do not use `px`, `rem`, or `%` for element sizing inside faces.
- **No new files unless necessary** — prefer editing existing files.
- **No documentation files** unless explicitly requested.

---
> Source: [urban-tech-creative/utc-website](https://github.com/urban-tech-creative/utc-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
