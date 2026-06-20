## glasskit-elements

> GlassKit Elements is a vanilla-JS Web Components library (v1.6.0) wrapping GlassKit CSS v1.6.0. It provides 29 custom elements with the `glk-` prefix, Dark/Light mode with automatic theme sync, Shadow DOM encapsulation, and form-associated custom elements. Use this reference whenever generating HTML that uses `<glk-*>` tags to ensure correct attributes, slots, events, and composition.


# GlassKit Elements – AI Component Reference

> **Purpose:** AI-optimized reference for generating correct `<glk-*>` markup.
> Companion to the class-based `SKILL.md` in [`@jungherz-de/glasskit`](https://github.com/JUNGHERZ/GlassKit) — the element wrappers delegate visuals and tokens to GlassKit CSS, so both references are best used together.

---

## 1. Setup & Boilerplate

### Installation

```bash
npm install @jungherz-de/glasskit-elements @jungherz-de/glasskit
```

Peer dependency `@jungherz-de/glasskit >=1.6.0` is required. The v1.6.0 floating Tab-Bar variant + Accessory capsule (`<glk-tab-dock>`, `<glk-tab-accessory>`, `<glk-tab-bar floating>`) requires CSS v1.6.0.

### Import (ES modules)

```js
// Full bundle — registers all 29 elements
import '@jungherz-de/glasskit-elements';

// Named imports (for direct references to constructor classes)
import { GlkButton, GlkModal } from '@jungherz-de/glasskit-elements';

// Tree-shaken per-component import
import { GlkButton } from '@jungherz-de/glasskit-elements/components/glk-button.js';
```

### CDN

```html
<script src="https://cdn.jsdelivr.net/npm/@jungherz-de/glasskit-elements/dist/glasskit-elements.min.js"></script>
```

Note: the elements bundle the GlassKit CSS `CSSStyleSheet` via `adoptedStyleSheets`. You do **not** need to load `glasskit.css` separately — it is already inside every component's shadow root.

### Minimal template

```html
<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.jsdelivr.net/npm/@jungherz-de/glasskit-elements/dist/glasskit-elements.min.js"></script>
</head>
<body>
  <div class="glass-bg">
    <glk-title>Hello GlassKit</glk-title>
    <glk-card>
      <p>Welcome!</p>
    </glk-card>
  </div>
</body>
</html>
```

The outer `.glass-bg` class (from GlassKit CSS) still provides the aurora background. Inside the shadow roots, each element carries its own glass styling.

### Theme

```html
<html data-theme="dark">  <!-- or "light" -->
```

```js
function toggleTheme() {
  const html = document.documentElement;
  const current = html.getAttribute('data-theme') || 'dark';
  html.setAttribute('data-theme', current === 'dark' ? 'light' : 'dark');
}
```

A single module-level `MutationObserver` watches `data-theme` on `<html>` and syncs every live GlassKit Element instance — you never set `data-theme` on individual elements.

---

## 2. Core Concepts

| Concept | Summary |
|---|---|
| Tag prefix | `glk-*` (analogous to the `glass-*` CSS prefix) |
| Rendering | Shadow DOM (`mode: 'open'`) with `adoptedStyleSheets` |
| Stylesheet sharing | GlassKit's `glassSheet` is the same `CSSStyleSheet` object in every element — no CSS duplication |
| Theme sync | Global MutationObserver on `<html data-theme>` |
| Events | Custom `glk-*` events, all `bubbles: true, composed: true` |
| Form participation | `GlkFormElement` uses `ElementInternals` (`static formAssociated = true`) |
| API style | Declarative HTML attributes + reflected JS properties |

Custom properties (`--gl-*`) defined on `:root` or `<html>` pass through shadow boundaries automatically, so custom theming works with a single global stylesheet.

---

## 3. Element Catalog (29 elements)

### 3.1 `<glk-nav>`

Horizontal navigation bar — flex container. Typically holds `<glk-pill>` buttons.

```html
<glk-nav>
  <glk-pill label="Back" onclick="history.back()">
    <svg viewBox="0 0 24 24"><polyline points="15 18 9 12 15 6"/></svg>
  </glk-pill>
  <glk-pill label="Settings">
    <svg viewBox="0 0 24 24">...</svg>
  </glk-pill>
</glk-nav>
```

No attributes. Default slot for children.

---

### 3.2 `<glk-pill>`

Circular icon button (46×46 px).

```html
<glk-pill label="Back">
  <svg viewBox="0 0 24 24"><polyline points="15 18 9 12 15 6"/></svg>
</glk-pill>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | `aria-label` for accessibility |
| `disabled` | Boolean | Disabled state |

Default slot: icon SVG (24×24).

Events: `glk-click` when clicked.

---

### 3.3 `<glk-tab-bar>` + `<glk-tab-item>`

Fixed bottom navigation. Requires `.glass-bg--has-tab-bar` on the outer background to reserve padding.

```html
<div class="glass-bg glass-bg--has-tab-bar">
  <!-- page content -->
  <glk-tab-bar>
    <glk-tab-item label="Home" active>
      <svg viewBox="0 0 24 24"><path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/></svg>
    </glk-tab-item>
    <glk-tab-item label="Search">
      <svg viewBox="0 0 24 24"><circle cx="11" cy="11" r="8"/></svg>
    </glk-tab-item>
    <glk-tab-item label="Profile" badge="3">
      <svg viewBox="0 0 24 24"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/></svg>
    </glk-tab-item>
  </glk-tab-bar>
</div>
```

**`<glk-tab-bar>`** — container.

| Attribute | Type | Description |
|---|---|---|
| `static` | Boolean | Switches from `position: fixed` to `position: relative` (useful for docs / previews) |
| `floating` | Boolean | Pill-shaped Liquid Glass variant — adds `.glass-tab-bar--floating`. Use inside `<glk-tab-dock>` |

**`<glk-tab-item>`**

| Attribute | Type | Description |
|---|---|---|
| `label` | String | Tab label |
| `active` | Boolean | Active state — gets a soft radial Spotlight halo when inside a `floating` bar |
| `badge` | String / Number | Numeric badge on the icon |

Default slot: icon SVG. Events: `glk-tab-select` on click.

#### Floating variant + Accessory (`<glk-tab-dock>` + `<glk-tab-accessory>`)

iOS 26 Liquid Glass-style floating tab bar. Wrap a `<glk-tab-bar floating>` in a `<glk-tab-dock>`, optionally with a `<glk-tab-accessory>` (search, compose…) capsule next to it. Use `.glass-bg--has-tab-bar-floating` on the outer background to reserve padding.

```html
<div class="glass-bg glass-bg--has-tab-bar-floating">
  <!-- page content -->
  <glk-tab-dock>
    <glk-tab-bar floating>
      <glk-tab-item label="Home" active>
        <svg viewBox="0 0 24 24"><path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/></svg>
      </glk-tab-item>
      <glk-tab-item label="Docs" badge="3">
        <svg viewBox="0 0 24 24"><rect x="2" y="3" width="20" height="14" rx="2"/></svg>
      </glk-tab-item>
      <glk-tab-item label="Upload">
        <svg viewBox="0 0 24 24"><path d="M12 16V4"/><polyline points="8 8 12 4 16 8"/></svg>
      </glk-tab-item>
    </glk-tab-bar>
    <glk-tab-accessory variant="accent" label="Compose">
      <svg viewBox="0 0 24 24"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
    </glk-tab-accessory>
  </glk-tab-dock>
</div>
```

**`<glk-tab-dock>`** — wrapper that holds the floating bar + optional accessory.

| Attribute | Type | Description |
|---|---|---|
| `accessory-left` | Boolean | Place the accessory on the left of the bar (default is right) |

**`<glk-tab-accessory>`** — standalone glass capsule next to the bar (e.g. search, compose). 56×56 px.

| Attribute | Type | Description |
|---|---|---|
| `label` | String | `aria-label` for accessibility |
| `variant` | String | `accent`, `success`, `error` — filled colored capsule with white icon |
| `disabled` | Boolean | Disabled state |

Default slot: icon SVG (22×22). Events: `glk-click` (suppressed when disabled).

---

### 3.4 `<glk-avatar>`

Inline avatar circle.

```html
<glk-avatar size="lg" src="/user.jpg"></glk-avatar>
<glk-avatar>MJ</glk-avatar>
```

| Attribute | Type | Description |
|---|---|---|
| `size` | String | `sm`, `md` (default), `lg` |
| `src` | String | Image URL; if omitted, text content is shown as initials |

---

### 3.5 `<glk-badge>`

Inline status badge.

```html
<glk-badge>Default</glk-badge>
<glk-badge variant="primary">Active</glk-badge>
<glk-badge variant="success">Done</glk-badge>
<glk-badge variant="error">Failed</glk-badge>
```

| Attribute | Type | Description |
|---|---|---|
| `variant` | String | `primary`, `success`, `error` |

Default slot: label text.

---

### 3.6 `<glk-card>`

Glass container for content.

```html
<glk-card>
  <p>Standard card content.</p>
</glk-card>

<glk-card glow>
  <p>Glowing card variant with light streak.</p>
</glk-card>
```

| Attribute | Type | Description |
|---|---|---|
| `glow` | Boolean | Adds frosted glass gradient + light streak |

Default slot: content.

---

### 3.7 `<glk-divider>`

Horizontal separator with gradient fade.

```html
<glk-divider></glk-divider>
```

No attributes, no content.

---

### 3.8 `<glk-status>`

Inline status notice (muted surface).

```html
<glk-status message="No results found"></glk-status>
```

| Attribute | Type | Description |
|---|---|---|
| `message` | String | Status text |

---

### 3.9 `<glk-title>`

Page title with text shadow.

```html
<glk-title>Settings</glk-title>
```

No attributes. Default slot: heading text.

---

### 3.10 `<glk-button>`

Full-width glass button (56 px). Three variants, four sizes.

```html
<glk-button variant="primary">Save</glk-button>
<glk-button variant="secondary">Cancel</glk-button>
<glk-button variant="tertiary">Learn more</glk-button>

<glk-button variant="primary" size="sm">Small</glk-button>
<glk-button variant="primary" size="lg">Large</glk-button>
<glk-button variant="primary" size="auto">Auto width</glk-button>

<glk-button variant="primary" disabled>Disabled</glk-button>
```

| Attribute | Type | Default | Description |
|---|---|---|---|
| `variant` | String | — | `primary`, `secondary`, `tertiary` |
| `size` | String | `md` | `sm`, `md`, `lg`, `auto` |
| `disabled` | Boolean | `false` | Disabled state |
| `type` | String | `button` | Native button type |

Events: `glk-click` (suppressed when disabled).

Default slot: button label (text or SVG + text).

---

### 3.11 `<glk-checkbox>`

Form-associated checkbox with glass styling.

```html
<glk-checkbox label="I agree to the terms" checked></glk-checkbox>
<glk-checkbox label="Subscribe to newsletter"></glk-checkbox>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | Label text |
| `checked` | Boolean | Checked state |
| `disabled` | Boolean | Disabled state |
| `name` | String | Form field name |
| `value` | String | Value submitted when checked (default `"on"`) |

Events: `glk-change` → `{ checked }`; also dispatches native `change`. Property: `.checked`.

---

### 3.12 `<glk-input>`

Text input with label + hint.

```html
<glk-input label="Email" type="email" placeholder="you@example.com" hint="We won't share."></glk-input>
<glk-input label="Password" type="password" required></glk-input>
<glk-input label="Broken field" error hint="This field is required."></glk-input>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | Label above the input |
| `type` | String | Native input type (`text`, `email`, `password`, `number`, …) |
| `placeholder` | String | Placeholder text |
| `hint` | String | Helper text below the input |
| `error` | Boolean | Error styling (red border + red hint) |
| `disabled` | Boolean | Disabled state |
| `required` | Boolean | Required field |
| `name` | String | Form field name |
| `value` | String | Current value |

Events: `glk-input` → `{ value }`, `glk-change` → `{ value }`. Native `input` / `change` also dispatched. Property: `.value`.

---

### 3.13 `<glk-radio>`

Radio button. Group multiple with the same `name`.

```html
<glk-radio name="plan" label="Free Plan" value="free" checked></glk-radio>
<glk-radio name="plan" label="Pro Plan" value="pro"></glk-radio>
<glk-radio name="plan" label="Enterprise" value="enterprise"></glk-radio>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | Label text |
| `name` | String | Group name |
| `value` | String | Value submitted when checked |
| `checked` | Boolean | Checked state |
| `disabled` | Boolean | Disabled state |

Events: `glk-change` → `{ checked, value }`.

---

### 3.14 `<glk-range>`

Slider input.

```html
<glk-range label="Volume" min="0" max="100" value="65" step="5"></glk-range>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | Label text |
| `min` | Number | Minimum value |
| `max` | Number | Maximum value |
| `value` | Number | Current value |
| `step` | Number | Step size |
| `name` | String | Form field name |
| `disabled` | Boolean | Disabled state |

Events: `glk-input`, `glk-change`. Property: `.value`.

---

### 3.15 `<glk-search>`

Search input with leading icon.

```html
<glk-search placeholder="Search components..."></glk-search>
```

| Attribute | Type | Description |
|---|---|---|
| `placeholder` | String | Placeholder text |
| `value` | String | Current value |
| `name` | String | Form field name |
| `disabled` | Boolean | Disabled state |

Events: `glk-input`, `glk-change`.

---

### 3.16 `<glk-select>`

Dropdown select. Pass native `<option>` elements as children.

```html
<glk-select label="Category" name="category">
  <option value="">Please select…</option>
  <option value="1">Insurance</option>
  <option value="2">Rental</option>
  <option value="3">Employment</option>
</glk-select>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | Label text |
| `name` | String | Form field name |
| `value` | String | Selected value |
| `disabled` | Boolean | Disabled state |

Children: native `<option>` elements (read from light DOM on initial render).

Events: `glk-change` → `{ value }`.

---

### 3.17 `<glk-textarea>`

Multi-line text input.

```html
<glk-textarea label="Message" rows="4" placeholder="Type your message..."></glk-textarea>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | Label text |
| `rows` | Number | Visible rows |
| `placeholder` | String | Placeholder text |
| `name` | String | Form field name |
| `value` | String | Current value |
| `disabled` | Boolean | Disabled state |
| `required` | Boolean | Required field |

Events: `glk-input`, `glk-change`.

---

### 3.18 `<glk-toggle>`

Switch-style toggle (form-associated).

```html
<glk-toggle label="Enable notifications" checked></glk-toggle>
<glk-toggle label="Dark mode"></glk-toggle>
<glk-toggle label="Disabled toggle" disabled></glk-toggle>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | Label text |
| `checked` | Boolean | Checked state |
| `disabled` | Boolean | Disabled state |
| `name` | String | Form field name |
| `value` | String | Submitted value when checked (default `"on"`) |

Events: `glk-change` → `{ checked }`. Property: `.checked`. Has `role="switch"` and syncs `aria-checked`.

---

### 3.19 `<glk-modal>`

Modal dialog with overlay, title, body, and footer actions.

```html
<glk-button variant="secondary" size="auto"
  onclick="document.getElementById('confirm').show()">
  Open
</glk-button>

<glk-modal id="confirm" title="Delete contract?">
  <p>This action cannot be undone.</p>
  <div slot="actions">
    <button class="glass-modal__action" onclick="document.getElementById('confirm').close()">Cancel</button>
    <button class="glass-modal__action glass-modal__action--danger"
            onclick="deleteContract(); document.getElementById('confirm').close()">Delete</button>
  </div>
</glk-modal>
```

| Attribute | Type | Description |
|---|---|---|
| `open` | Boolean | Visibility state (reflected as `.is-active` on the overlay) |
| `title` | String | Header title |

Slots:
- default — modal body content
- `actions` — container for footer buttons (GlassKit expects `<button class="glass-modal__action">`)

Methods: `.show()`, `.close()`. Property: `.open`. Events: `glk-close`.

Closes automatically on overlay click and <kbd>Escape</kbd>.

---

### 3.20 `<glk-popover>`

Anchored dropdown / menu container. Wraps a trigger element (`slot="trigger"`) and floating content. The element handles `.is-open` toggling, outside-click dismiss, and <kbd>Escape</kbd>-key close internally.

```html
<glk-popover placement="end">
  <glk-button slot="trigger" variant="secondary" size="auto">Open menu</glk-button>
  <glk-list bare>
    <glk-list-item title="Share" interactive></glk-list-item>
    <glk-list-item title="Duplicate" interactive></glk-list-item>
    <glk-list-item title="Delete" variant="danger" interactive></glk-list-item>
  </glk-list>
</glk-popover>
```

| Attribute | Type | Default | Description |
|---|---|---|---|
| `open` | Boolean | `false` | Visibility state |
| `placement` | String | `bottom` | `top`, `bottom`, `start`, `end` |

Slots:
- `trigger` — element that toggles the popover on click
- default — floating content shown while open

Methods: `.show()`, `.close()`, `.toggle()`. Property: `.open`.
Events: `glk-open`, `glk-close`.

**Naming warning:** the toggle method is deliberately called `.toggle()`, **not** `.togglePopover()` — the latter collides with the native `HTMLElement.togglePopover()` API from the HTML Popover specification.

---

### 3.21 `<glk-progress>`

Progress bar.

```html
<glk-progress label="Upload" value="75" variant="success"></glk-progress>
<glk-progress label="Storage" value="90" variant="error" size="lg"></glk-progress>
<glk-progress label="CPU" value="45" size="sm"></glk-progress>
```

| Attribute | Type | Description |
|---|---|---|
| `label` | String | Label text shown left of the bar |
| `value` | Number | 0–100 percentage |
| `variant` | String | `success`, `error` |
| `size` | String | `sm`, `md` (default), `lg` |

---

### 3.22 `<glk-toast>`

Auto-dismissing notification popover (shown imperatively).

```html
<glk-toast id="toast"></glk-toast>

<script>
  document.getElementById('toast').show('Saved successfully!', 'success', 3000);
</script>
```

| Attribute | Type | Description |
|---|---|---|
| `variant` | String | `success`, `error`, `warning` |
| `duration` | Number | Auto-dismiss time in ms (default `3000`) |

Methods: `.show(message, variant, duration)`, `.dismiss()`.

---

### 3.23 `<glk-accordion>` + `<glk-accordion-item>`

Collapsible sections.

```html
<glk-accordion>
  <glk-accordion-item title="What is GlassKit?" open>
    A glassmorphism CSS component library.
  </glk-accordion-item>
  <glk-accordion-item title="Do I need a framework?">
    No — vanilla HTML works.
  </glk-accordion-item>
</glk-accordion>
```

**`<glk-accordion>`** — container, no attributes.

**`<glk-accordion-item>`**

| Attribute | Type | Description |
|---|---|---|
| `title` | String | Trigger label |
| `open` | Boolean | Expanded state |

Default slot: body content. Events: `glk-toggle` → `{ open }`. Property: `.open`.

---

### 3.24 `<glk-list>` + `<glk-list-item>`

iOS-style grouped settings list. Items carry a leading icon, title + optional subtitle, and a trailing element. Dividers between items are drawn automatically by GlassKit CSS — **never** add `<hr>` or divider markup manually. Supports section headers, large icons (40×40), multi-line subtitles, trailing value text, and semantic `danger`/`accent` variants.

#### Settings-style list with section header and large icons

```html
<glk-list header="Recommendations">
  <glk-list-item title="Review downloaded media" subtitle="Save up to 1.38 GB. Review files and remove as needed." leading-lg wrap interactive>
    <svg slot="leading" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
      <polyline points="7 10 12 15 17 10"/>
      <line x1="12" y1="15" x2="12" y2="3"/>
    </svg>
    <svg slot="trailing" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <polyline points="9 18 15 12 9 6"/>
    </svg>
  </glk-list-item>
  <glk-list-item title="AusweisApp Bund" subtitle="Version 2.5.1" leading-lg detail="24 MB" interactive>
    <svg slot="leading" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <rect x="2" y="3" width="20" height="18" rx="2"/>
      <path d="M9 3v18"/><path d="M15 3v18"/>
    </svg>
    <svg slot="trailing" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <polyline points="9 18 15 12 9 6"/>
    </svg>
  </glk-list-item>
  <glk-list-item title="View all subscriptions" variant="accent" center interactive></glk-list-item>
</glk-list>
```

#### Compact menu with danger variant

```html
<glk-list>
  <glk-list-item title="Share" center interactive></glk-list-item>
  <glk-list-item title="Duplicate" center interactive></glk-list-item>
  <glk-list-item title="Delete" variant="danger" center interactive></glk-list-item>
</glk-list>
```

#### Attributes

**`<glk-list>`**

| Attribute | Type | Description |
|---|---|---|
| `header` | String | Section header text — renders as uppercase label above the list |
| `flush` | Boolean | Edge-to-edge variant — removes side margin and radius |
| `bare` | Boolean | Strips background, border, shadow — use inside `<glk-popover>` or `<glk-card>` to avoid double glass surfaces |

**`<glk-list-item>`**

| Attribute | Type | Description |
|---|---|---|
| `title` | String | Primary text (medium weight, truncated with ellipsis) |
| `subtitle` | String | Secondary text (muted, small, truncated) |
| `interactive` | Boolean | Enables hover / focus / active states and `glk-click` event |
| `center` | Boolean | Centered single-text variant (for action rows like "Reset to defaults") |
| `leading-lg` | Boolean | Large 40×40 leading icon with rounded corners (for app icons; use 32×32 SVG or `<img>`) |
| `wrap` | Boolean | Multi-line subtitle — allows up to 3 lines with ellipsis (requires `subtitle`) |
| `detail` | String | Muted trailing value text (e.g. file size "24 MB", version number) — rendered before `slot="trailing"` |
| `variant` | String | Semantic color: `"danger"` (red destructive) or `"accent"` (primary color). Inherits to title and leading icon |

#### Slots (on `<glk-list-item>`)

| Slot | Purpose |
|---|---|
| `leading` | Icon slot (24×24 SVG recommended, or 32×32 with `leading-lg`; use `stroke: currentColor`) |
| `trailing` | Chevron, badge, or button — rendered after `detail` value |

#### Events

`<glk-list-item>` emits `glk-click` when clicked, but only if `interactive` is set.

#### Internals (for framework authors)

Both elements use pure Shadow DOM with slot projection — no child node cloning — so they compose cleanly inside lit-html, HybridsJS, React, Vue, and Svelte templates. `<glk-list>` observes its direct children via `MutationObserver` and sets a `data-last` attribute on the final `<glk-list-item>`; each item's shadow carries a one-line override sheet (`:host([data-last]) .glass-list__item::after { content: none; }`) to suppress the auto-divider on the last row. Divider inset automatically adjusts for `leading-lg` via CSS `:has()`. Zero CSS duplication.

---

## 4. Composition Patterns

### Login Screen

```html
<div class="glass-bg">
  <glk-nav>
    <glk-pill label="Back" onclick="history.back()">
      <svg viewBox="0 0 24 24"><polyline points="15 18 9 12 15 6"/></svg>
    </glk-pill>
    <glk-pill label="Toggle theme" onclick="toggleTheme()">
      <svg class="icon-moon" viewBox="0 0 24 24"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/></svg>
    </glk-pill>
  </glk-nav>

  <glk-title>Sign In</glk-title>

  <form>
    <glk-input label="Email" type="email" name="email" placeholder="you@example.com" required></glk-input>
    <glk-input label="Password" type="password" name="password" required></glk-input>

    <glk-button variant="primary" type="submit">Log In</glk-button>
    <glk-button variant="tertiary">Forgot password?</glk-button>
  </form>
</div>
```

### Dashboard with Cards + Tab Bar

```html
<div class="glass-bg glass-bg--has-tab-bar">
  <glk-nav>
    <glk-title>Dashboard</glk-title>
  </glk-nav>

  <glk-card glow>
    <p>Capture and upload documents.</p>
  </glk-card>
  <glk-card glow>
    <p>Manage and archive contracts.</p>
  </glk-card>

  <glk-tab-bar>
    <glk-tab-item label="Home" active>
      <svg viewBox="0 0 24 24">...</svg>
    </glk-tab-item>
    <glk-tab-item label="Contracts" badge="3">
      <svg viewBox="0 0 24 24">...</svg>
    </glk-tab-item>
    <glk-tab-item label="Profile">
      <svg viewBox="0 0 24 24">...</svg>
    </glk-tab-item>
  </glk-tab-bar>
</div>
```

### Form Page with Validation

```html
<form id="contract-form">
  <glk-title>New Contract</glk-title>

  <glk-input label="Contract Name" name="name" required></glk-input>
  <glk-select label="Category" name="category">
    <option value="">Please select…</option>
    <option value="insurance">Insurance</option>
    <option value="rental">Rental</option>
  </glk-select>
  <glk-textarea label="Notes" name="notes" rows="4"></glk-textarea>
  <glk-toggle label="Enable reminder" name="reminder"></glk-toggle>

  <glk-button variant="primary" type="submit">Save</glk-button>
</form>

<script>
  document.getElementById('contract-form').addEventListener('submit', (e) => {
    e.preventDefault();
    const data = new FormData(e.target);
    console.log(Object.fromEntries(data));  // { name, category, notes, reminder? }
  });
</script>
```

### Delete Confirmation Modal

```html
<glk-button variant="secondary" size="auto"
  onclick="document.getElementById('confirm').show()">
  Delete contract
</glk-button>

<glk-modal id="confirm" title="Delete contract?">
  <p>This action cannot be undone.</p>
  <div slot="actions">
    <button class="glass-modal__action" onclick="document.getElementById('confirm').close()">Cancel</button>
    <button class="glass-modal__action glass-modal__action--danger"
            onclick="handleDelete(); document.getElementById('confirm').close()">Delete</button>
  </div>
</glk-modal>
```

### iOS-style Settings Screen (List + Popover)

Full example reproducing the native iOS Settings layout with two grouped lists and an inline action menu.

```html
<div class="glass-bg">
  <glk-nav>
    <glk-pill label="Back" onclick="history.back()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="15 18 9 12 15 6"/></svg>
    </glk-pill>
    <glk-title>Settings</glk-title>

    <glk-popover placement="end">
      <glk-pill slot="trigger" label="More">
        <svg viewBox="0 0 24 24" fill="currentColor"><circle cx="12" cy="5" r="1.5"/><circle cx="12" cy="12" r="1.5"/><circle cx="12" cy="19" r="1.5"/></svg>
      </glk-pill>
      <glk-list bare>
        <glk-list-item title="Edit profile" interactive></glk-list-item>
        <glk-list-item title="Sign out" interactive></glk-list-item>
      </glk-list>
    </glk-popover>
  </glk-nav>

  <glk-list header="Account">
    <glk-list-item title="Marcel Jungherz" subtitle="marcel@jungherz.com" leading-lg interactive>
      <svg slot="leading" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
      <svg slot="trailing" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="9 18 15 12 9 6"/></svg>
    </glk-list-item>
    <glk-list-item title="Subscriptions" interactive detail="$34.94">
      <svg slot="leading" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="5" width="20" height="14" rx="2"/><line x1="2" y1="10" x2="22" y2="10"/></svg>
      <svg slot="trailing" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="9 18 15 12 9 6"/></svg>
    </glk-list-item>
  </glk-list>

  <glk-list header="Notifications">
    <glk-list-item title="Push notifications" interactive>
      <svg slot="leading" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M18 8a6 6 0 1 0-12 0c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/></svg>
      <span slot="trailing">On</span>
    </glk-list-item>
    <glk-list-item title="Email digest" interactive>
      <svg slot="leading" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22 6 12 13 2 6"/></svg>
      <span slot="trailing">Weekly</span>
    </glk-list-item>
    <glk-list-item title="Reset to defaults" variant="danger" center interactive></glk-list-item>
  </glk-list>
</div>
```

Key points:
- Section headers via `header` attribute — no need for external `<p>` labels
- Large leading icon via `leading-lg` for profile-style rows
- Trailing value via `detail` attribute — muted metadata text before the chevron
- Auto-dividers — no manual `<hr>` between rows; inset adapts for large icons automatically
- `variant="danger"` on destructive "Reset" row — red text signals caution
- Popover anchored in nav — `placement="end"` aligns the menu to the right edge of the trigger
- Bare list inside popover — `<glk-list bare>` strips the inner glass surface so only the popover's glass shows

---

## 5. State & Event Overview

| Element | Key Property | HTML Attribute | Event(s) | Detail |
|---|---|---|---|---|
| `<glk-button>` | — | `disabled` | `glk-click` | — |
| `<glk-pill>` | — | `disabled` | `glk-click` | — |
| `<glk-tab-item>` | — | `active` | `glk-tab-select` | — |
| `<glk-tab-accessory>` | — | `disabled` | `glk-click` | — |
| `<glk-input>` | `.value` | `value` | `glk-input`, `glk-change` (+ native `input`, `change`) | `{ value }` |
| `<glk-textarea>` | `.value` | `value` | `glk-input`, `glk-change` | `{ value }` |
| `<glk-select>` | `.value` | `value` | `glk-change` | `{ value }` |
| `<glk-search>` | `.value` | `value` | `glk-input`, `glk-change` | `{ value }` |
| `<glk-toggle>` | `.checked` | `checked` | `glk-change` (+ native `change`) | `{ checked }` |
| `<glk-checkbox>` | `.checked` | `checked` | `glk-change` | `{ checked }` |
| `<glk-radio>` | `.checked` | `checked` | `glk-change` | `{ checked, value }` |
| `<glk-range>` | `.value` | `value` | `glk-input`, `glk-change` | `{ value }` |
| `<glk-modal>` | `.open` | `open` | `glk-close` | — |
| `<glk-popover>` | `.open` | `open` | `glk-open`, `glk-close` | — |
| `<glk-accordion-item>` | `.open` | `open` | `glk-toggle` | `{ open }` |
| `<glk-list-item>` | — | `interactive` | `glk-click` (only when interactive) | — |
| `<glk-toast>` | — | — | — | (imperative) |

All `glk-*` events bubble and are `composed: true`, so they pierce shadow boundaries naturally.

---

## 6. Rules & Common Mistakes

### Always follow

1. **`data-theme` on `<html>`** — never on `<body>` or individual elements; the global MutationObserver only watches `<html>`.
2. **Form elements belong inside a `<form>`** — otherwise `ElementInternals.setFormValue()` has no effect and `new FormData()` will be empty.
3. **`<glk-list-item>` must be a direct child of `<glk-list>`** — nested deeper, it won't be observed for the auto-divider last-item detection.
4. **Popover trigger via `slot="trigger"`** — don't reference external elements by ID. The popover's built-in click handler listens on the trigger slot.
5. **Toggle popovers through the API** — `popover.open = true` or `popover.toggle()`, never `classList.add('is-open')` on internal nodes.
6. **Use `<glk-list bare>` inside `<glk-popover>`** — otherwise you get a double glass surface.
7. **Import the library once** — `import '@jungherz-de/glasskit-elements'` registers all elements via `customElements.define`. Multiple imports are idempotent but unnecessary.
8. **SVG icons: `stroke: currentColor, stroke-width: 2`** — matches GlassKit's icon convention and inherits the token-driven text color.

### Common mistakes

| Mistake | Correction |
|---|---|
| `onclick="popover.togglePopover()"` | Collides with the native `HTMLElement.togglePopover()` method (HTML Popover API). Use `.toggle()` on the `<glk-popover>` element, or set the `open` attribute. |
| `<hr>` or `<glk-divider>` manually between `<glk-list-item>`s | Remove it — dividers render automatically via `::after`. |
| `<glk-list>` inside `<glk-popover>` without `bare` | Add `bare` to the inner list so only the popover surface has glass. |
| `document.querySelector('.glass-modal').classList.add('is-active')` | Set `<glk-modal>.open = true` instead — the element manages `.is-active` internally. |
| `data-theme` directly on a `<glk-*>` element | Set it on `<html>`; theme propagates automatically through the MutationObserver. |
| Form element outside `<form>` | Wrap in a `<form>`; `ElementInternals` needs a form owner. |
| Overriding shadow styles with external CSS | Use CSS custom properties (`--gl-color-primary`, etc.) — they pierce shadow roots. Hard-coded selectors inside shadow roots cannot be targeted externally. |
| Forgetting `slot="trigger"` on the popover trigger element | Without it, the popover's internal click handler can't identify the trigger and never toggles. |
| Using `open` as a method name (`mypopover.open()`) | `open` is a reflected property. Use `.show()` / `.close()` / `.toggle()` for imperative control. |
| `<glk-tab-bar>` without `.glass-bg--has-tab-bar` on the outer background | Tab bar covers bottom content. Add the modifier to the outer `.glass-bg` container. |
| `<glk-tab-bar floating>` without `<glk-tab-dock>` wrapper | Floating bar relies on the dock for fixed centered positioning. Always wrap it in `<glk-tab-dock>`. |
| `<glk-tab-dock>` without `.glass-bg--has-tab-bar-floating` on the outer background | Use `--has-tab-bar-floating` (not `--has-tab-bar`) for the floating variant — the padding budget differs. |

---

## 7. Quick Reference

| Tag | Category | Key attributes | Key slots | Key events |
|---|---|---|---|---|
| `<glk-nav>` | Navigation | — | default | — |
| `<glk-pill>` | Navigation | `label`, `disabled` | default (icon) | `glk-click` |
| `<glk-tab-bar>` | Navigation | `static`, `floating` | default | — |
| `<glk-tab-item>` | Navigation | `label`, `active`, `badge` | default (icon) | `glk-tab-select` |
| `<glk-tab-dock>` | Navigation | `accessory-left` | default | — |
| `<glk-tab-accessory>` | Navigation | `label`, `variant`, `disabled` | default (icon) | `glk-click` |
| `<glk-avatar>` | Content | `size`, `src` | default (initials) | — |
| `<glk-badge>` | Content | `variant` | default | — |
| `<glk-card>` | Content | `glow` | default | — |
| `<glk-divider>` | Content | — | — | — |
| `<glk-status>` | Content | `message` | — | — |
| `<glk-title>` | Content | — | default | — |
| `<glk-button>` | Buttons | `variant`, `size`, `disabled`, `type` | default | `glk-click` |
| `<glk-checkbox>` | Forms | `label`, `checked`, `disabled`, `name`, `value` | — | `glk-change` |
| `<glk-input>` | Forms | `label`, `type`, `placeholder`, `hint`, `error`, `disabled`, `required`, `name`, `value` | — | `glk-input`, `glk-change` |
| `<glk-radio>` | Forms | `label`, `name`, `value`, `checked`, `disabled` | — | `glk-change` |
| `<glk-range>` | Forms | `label`, `min`, `max`, `value`, `step`, `name`, `disabled` | — | `glk-input`, `glk-change` |
| `<glk-search>` | Forms | `placeholder`, `value`, `name`, `disabled` | — | `glk-input`, `glk-change` |
| `<glk-select>` | Forms | `label`, `name`, `value`, `disabled` | default (`<option>`) | `glk-change` |
| `<glk-textarea>` | Forms | `label`, `rows`, `placeholder`, `name`, `value`, `disabled`, `required` | — | `glk-input`, `glk-change` |
| `<glk-toggle>` | Forms | `label`, `checked`, `disabled`, `name`, `value` | — | `glk-change` |
| `<glk-modal>` | Feedback | `open`, `title` | default, `actions` | `glk-close` |
| `<glk-popover>` | Feedback | `open`, `placement` | `trigger`, default | `glk-open`, `glk-close` |
| `<glk-progress>` | Feedback | `value`, `label`, `variant`, `size` | — | — |
| `<glk-toast>` | Feedback | `variant`, `duration` | — | (imperative) |
| `<glk-accordion>` | Containers | — | default | — |
| `<glk-accordion-item>` | Containers | `title`, `open` | default | `glk-toggle` |
| `<glk-list>` | Containers | `header`, `flush`, `bare` | default | — |
| `<glk-list-item>` | Containers | `title`, `subtitle`, `interactive`, `center`, `leading-lg`, `wrap`, `detail`, `variant` | `leading`, `trailing` | `glk-click` (when interactive) |

---

## 8. Framework Integration

GlassKit Elements are standards-compliant Custom Elements. They work in every major framework:

### Vanilla HTML / JS

```html
<script type="module" src="https://cdn.jsdelivr.net/npm/@jungherz-de/glasskit-elements/dist/glasskit-elements.esm.js"></script>
<glk-button variant="primary" onclick="alert('clicked')">Click</glk-button>
```

### React

```jsx
import '@jungherz-de/glasskit-elements';

function App() {
  return (
    <glk-input
      label="Name"
      value={name}
      onGlkInput={(e) => setName(e.detail.value)}
    />
  );
}
```

For React 18, custom-event listeners use the lowercase `onglkchange` form, or attach manually via `ref` + `addEventListener`. React 19 supports `onGlkChange`-style camelCase directly.

### Vue 3

```vue
<template>
  <glk-input
    label="Name"
    :value="name"
    @glk-input="e => name = e.detail.value"
  />
</template>

<script setup>
import '@jungherz-de/glasskit-elements';
import { ref } from 'vue';
const name = ref('');
</script>
```

### Svelte

```svelte
<script>
  import '@jungherz-de/glasskit-elements';
  let name = '';
</script>

<glk-input
  label="Name"
  value={name}
  on:glk-input={e => name = e.detail.value}
/>
```

### lit-html / Hybrids

```js
import { html } from 'lit-html';
import '@jungherz-de/glasskit-elements';

const template = (items) => html`
  <glk-list>
    ${items.map(item => html`
      <glk-list-item title=${item.title} subtitle=${item.subtitle} ?interactive=${true}>
        ${item.icon ? html`<svg slot="leading">...</svg>` : ''}
      </glk-list-item>
    `)}
  </glk-list>
`;
```

Framework-safe note: `<glk-list>` and `<glk-list-item>` use pure slot projection — no child cloning — so lit-html parts, Hybrids templates, and React virtual-DOM bindings stay intact across re-renders.

### SSR

GlassKit Elements are currently **client-side only** — they rely on `document.documentElement`, `adoptedStyleSheets`, and `ElementInternals` which are unavailable during server-side rendering. In SSR frameworks (Next.js, Nuxt, SvelteKit, Astro), use client-only directives or dynamic imports to defer hydration.

---

## 9. Custom Theming

Theming works identically to GlassKit CSS — define custom-property overrides in your global stylesheet; they pass through shadow boundaries automatically:

```css
:root {
  --gl-color-primary:      #007AFF;
  --gl-color-primary-dark: #0055CC;
  --gl-color-primary-mid:  #0066E0;

  --gl-border-warm:        rgba(0, 122, 255, 0.35);
  --gl-border-focus:       rgba(0, 122, 255, 0.60);

  --gl-shadow-btn-primary: 0 6px 24px rgba(0, 100, 220, 0.35),
                            0 2px 8px rgba(0, 0, 0, 0.15);
  --gl-shadow-focus:       0 0 0 3px rgba(0, 122, 255, 0.3);
}
```

See the class-based [GlassKit CSS `SKILL.md`](https://github.com/JUNGHERZ/GlassKit/blob/main/SKILL.md) Section 2 for the complete list of design tokens (colors, surfaces, borders, blur, radii, spacing, shadows, typography).

---

## 10. Architecture Notes

| Concept | Location |
|---|---|
| Base class | `src/base.js` → `GlkElement`, `GlkFormElement` |
| Theme sync | Single module-level `MutationObserver` in `base.js` |
| Adopted stylesheets | `glassSheet` (from `@jungherz-de/glasskit/glasskit-styles.js`) + module-level `hostSheet` / `inlineHostSheet` |
| Per-component structure | One `.js` file per element in `src/components/{category}/glk-{name}.js` |
| Barrel | `src/index.js` — exports and registers all 27 elements |
| Build | Rollup → IIFE (`glasskit-elements.js`), minified IIFE, and ESM (`glasskit-elements.esm.js`) |

Each element is a subclass of `GlkElement` (or `GlkFormElement` for form controls) and follows a consistent lifecycle:

1. `constructor()` — attach open shadow, adopt GlassKit stylesheet + host display sheet
2. `connectedCallback()` — create theme wrapper, call `render()`, call `setupEvents()`, register for global theme sync
3. `render()` — build shadow DOM tree (override per component)
4. `setupEvents()` — attach DOM listeners (override per component)
5. `attributeChangedCallback()` — call `onAttributeChanged()` (override per component)
6. `disconnectedCallback()` — unregister, call `teardownEvents()`

The `GlkFormElement` subclass adds `ElementInternals` for native form participation via `this.setFormValue(value)`, plus `formResetCallback` → `resetValue()` and `formStateRestoreCallback` → `restoreValue()`.

---
> Source: [JUNGHERZ/GlassKit-Elements](https://github.com/JUNGHERZ/GlassKit-Elements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
