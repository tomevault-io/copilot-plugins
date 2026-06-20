## buefy

> Official Buefy skill for Vue.js + Bulma UI components. TRIGGER when generating any Vue template, SFC, or component code that involves UI elements, forms, modals, tables, navigation, or feedback messages.


# Buefy

Buefy is a lightweight UI component library for Vue 3 based on Bulma CSS. Components are Vue SFCs (`<b-button>`, `<b-modal>`, etc.) that wrap Bulma's class-based styles with reactive props, events, and slots.

## When to run this skill

Trigger whenever you are:
- Writing a Vue template or SFC that includes any UI element (buttons, forms, modals, tables, navigation, etc.)
- Asked to "use Buefy" or "add a Buefy component"
- Working in a project that has `buefy` in its `package.json` dependencies
- Generating component code without a specified UI library in a Vue project

## Install

```bash
npm install buefy
```

### Full bundle (registers all components globally)

```typescript
// main.ts
import { createApp } from 'vue'
import Buefy from 'buefy'
import 'buefy/dist/buefy.css'
import App from './App.vue'

const app = createApp(App)
app.use(Buefy)
app.mount('#app')
```

### Individual components (tree-shakeable)

```typescript
// main.ts — register as plugins
import { createApp } from 'vue'
import { Button, Field, Input } from 'buefy'
import App from './App.vue'

const app = createApp(App)
app.use(Button)
app.use(Field)
app.use(Input)
app.mount('#app')
```

```vue
<!-- Or import directly in a SFC -->
<script lang="ts">
import { BButton, BField, BInput } from 'buefy'
export default { components: { BButton, BField, BInput } }
</script>
```

## Usage rules

1. **Use the `type` prop, not `class`** — Buefy components expose Bulma modifiers as a `type` prop. Use `type="is-primary"` not `class="is-primary"` on Buefy components.
2. **Size prop values** — `"is-small"` | `"is-medium"` | `"is-large"`. No bare size strings.
3. **Always wrap form inputs in `<b-field>`** — `<b-field>` provides the label, validation message, and grouping. Direct bare inputs miss label/message wiring.
4. **v-model target** — Most components bind to `modelValue`. Named v-models on `b-table`: `v-model:selected`, `v-model:checked-rows`, `v-model:current-page`, `v-model:opened-detailed`.
5. **Icons** — Use the `icon` prop with the icon name only (no `mdi-` prefix). The icon pack (MDI by default) must be registered in your project. Example: `icon="home"` not `icon="mdi-home"`.
6. **b-table columns** — `<b-table>` requires `:data="arrayOfObjects"` and `<b-table-column field="key" label="Header">` children. Columns can also be passed via the `:columns` prop.
7. **Theming** — Customize via Bulma SCSS variable overrides in a `.scss` file, not a JS config object. The `ConfigProgrammatic` utility sets defaults (icon pack, locale, etc.).
8. **Programmatic APIs** — Components like Toast, Snackbar, Dialog, Loading, Modal, and Notification can be opened imperatively. Import the programmatic class or use the composable:
   ```typescript
   import { ToastProgrammatic as Toast } from 'buefy'
   Toast.open('Hello world')
   // or with options:
   Toast.open({ message: 'Saved!', type: 'is-success', duration: 3000 })

   // Composable-style (preferred in Composition API):
   import { useToast } from 'buefy'
   const { open: openToast } = useToast()
   openToast({ message: 'Saved!', type: 'is-success' })
   ```

## Colors & Types

Use these values for the `type` prop on components that accept it:

| Value | Meaning |
|-------|---------|
| `is-primary` | Primary brand color (default blue) |
| `is-link` | Link color |
| `is-info` | Informational (cyan) |
| `is-success` | Success (green) |
| `is-warning` | Warning (yellow) |
| `is-danger` | Error / danger (red) |
| `is-dark` | Dark (near-black) |
| `is-light` | Light (near-white) |
| `is-white` | White |
| `is-black` | Black |

Size values: `is-small` · `is-medium` · `is-large`

---

## Components

### b-autocomplete

Autocomplete text input with dropdown suggestions.

**Key props:** `v-model` (String), `data` (Array — suggestion list), `field` (String — dot path into data objects for display), `expanded`, `loading`, `icon`, `size`, `rounded`, `keep-first`, `open-on-focus`, `keep-open`, `clearable`, `placeholder`

**Slots:** default (scoped: `{ option, index }`) for custom option rendering; `header`, `footer`, `empty`, `group`

**Emits:** `typing`, `select`, `focus`, `blur`, `infinite-scroll`

```vue
<b-field label="Search">
  <b-autocomplete
    v-model="search"
    :data="filteredResults"
    field="name"
    placeholder="Type to search"
    @typing="onTyping"
    @select="onSelect"
  >
    <template #default="{ option }">
      {{ option.name }} — {{ option.category }}
    </template>
  </b-autocomplete>
</b-field>
```

---

### b-breadcrumb

Breadcrumb navigation trail.

**Key props:** `align` (`"is-left"` | `"is-centered"` | `"is-right"`), `separator` (`"has-arrow-separator"` | `"has-bullet-separator"` | `"has-dot-separator"` | `"has-succeeds-separator"`), `size`

**Slots:** default — place `<b-breadcrumb-item>` children

```vue
<b-breadcrumb>
  <b-breadcrumb-item tag="router-link" to="/">Home</b-breadcrumb-item>
  <b-breadcrumb-item tag="router-link" to="/docs">Docs</b-breadcrumb-item>
  <b-breadcrumb-item active>Current Page</b-breadcrumb-item>
</b-breadcrumb>
```

---

### b-button

Standard button with Bulma styling.

**Key props:** `type`, `size`, `label` (String — overrides slot), `icon-left`, `icon-right`, `icon-pack`, `loading`, `outlined`, `expanded`, `rounded`, `inverted`, `native-type` (`"button"` | `"submit"` | `"reset"`), `tag` (renders as different element/component)

**Slots:** default (button content when `label` not set)

```vue
<b-button type="is-primary" icon-left="plus" @click="add">Add Item</b-button>
<b-button type="is-danger" outlined loading>Deleting…</b-button>
<b-button tag="router-link" to="/about" type="is-link">About</b-button>
```

---

### b-carousel

Horizontally-scrolling image/content carousel.

**Key props:** `v-model` (Number — active slide index), `animated` (`"slide"` | `"fade"`), `interval` (Number ms, autoplay speed), `autoplay`, `pause-hover`, `arrow`, `arrow-hover`, `indicator`, `indicator-style`, `overlay`, `progress`, `progress-type`, `icon-pack`, `icon-prev`, `icon-next`

**Slots:** default (place `<b-carousel-item>` children); `list` (scoped: `{ active, switch }`) — custom indicator list; `indicators` (scoped: `{ i }`)

```vue
<b-carousel v-model="activeSlide" :autoplay="false">
  <b-carousel-item v-for="(item, i) in items" :key="i">
    <figure class="image is-16by9">
      <img :src="item.image" :alt="item.alt">
    </figure>
  </b-carousel-item>
</b-carousel>
```

---

### b-checkbox

Checkbox input.

**Key props:** `v-model` (Boolean | Array), `native-value` (value pushed into array when checked), `type`, `size`, `disabled`, `required`, `name`, `indeterminate`, `true-value`, `false-value`

**Slots:** default (label text)

```vue
<!-- Single boolean -->
<b-checkbox v-model="agreed">I agree to the terms</b-checkbox>

<!-- Array binding — checked items pushed into list -->
<b-checkbox v-model="selected" native-value="apple">Apple</b-checkbox>
<b-checkbox v-model="selected" native-value="banana">Banana</b-checkbox>
```

---

### b-checkbox-button

Checkbox styled as a button (use inside a `<b-field grouped>`).

**Key props:** Same as `b-checkbox` plus `type`

```vue
<b-field grouped>
  <b-checkbox-button v-model="selected" native-value="A" type="is-primary">Option A</b-checkbox-button>
  <b-checkbox-button v-model="selected" native-value="B" type="is-primary">Option B</b-checkbox-button>
</b-field>
```

---

### b-clockpicker

Visual clock face time picker (alternative to b-timepicker).

**Key props:** `v-model` (Date), `placeholder`, `inline`, `editable`, `disabled`, `size`, `expanded`, `icon`, `icon-pack`, `rounded`, `loading`, `hour-format` (`"12"` | `"24"`), `increment-minutes`, `min-time`, `max-time`

**Slots:** `trigger`

```vue
<b-field label="Time">
  <b-clockpicker v-model="time" placeholder="Select time" hour-format="12" />
</b-field>
```

---

### b-collapse

Collapsible content section.

**Key props:** `v-model` (Boolean — open state), `animation`, `position` (`"is-top"` | `"is-bottom"`)

**Slots:** `trigger` (scoped: `{ open }`) — the toggle trigger element; default — the collapsible content

**Emits:** `open`, `close`

```vue
<b-collapse v-model="isOpen">
  <template #trigger="{ open }">
    <b-button :label="open ? 'Hide' : 'Show'" :icon-left="open ? 'chevron-up' : 'chevron-down'" />
  </template>
  <div class="notification">
    This content is collapsible.
  </div>
</b-collapse>
```

---

### b-colorpicker

HSL color picker with dropdown.

**Key props:** `v-model` (Color object `{ red, green, blue, alpha }`), `position`, `inline`, `expanded`, `disabled`, `alpha` (show alpha slider), `representation` (`"square"` | `"triangle"`)

**Slots:** `trigger`, `header`, `footer` (scoped: `{ color }`)

```vue
<b-field label="Pick color">
  <b-colorpicker v-model="color" />
</b-field>
```

---

### b-datepicker

Date (or date range) picker with calendar dropdown.

**Key props:** `v-model` (Date | Date[]), `min-date`, `max-date`, `placeholder`, `inline`, `editable`, `disabled`, `range`, `multiple`, `size`, `expanded`, `icon`, `icon-pack`, `rounded`, `loading`, `type` (`"month"` for month-only picker), `date-formatter`, `date-parser`, `locale`

**Slots:** `trigger`, `header`, `footer`

```vue
<b-field label="Date">
  <b-datepicker v-model="date" placeholder="Click to select…" icon="calendar" />
</b-field>

<!-- Range picker -->
<b-datepicker v-model="dateRange" range />
```

---

### b-datetimepicker

Combined date + time picker.

**Key props:** All `b-datepicker` props, plus `datetime-formatter`, `datetime-parser`, `time-creator`, and timepicker pass-through via `:timepicker`

```vue
<b-field label="Appointment">
  <b-datetimepicker v-model="datetime" placeholder="Select date and time" />
</b-field>
```

---

### b-dialog

Programmatic confirmation/alert/prompt dialog (not a declarative component).

**Open imperatively:**

```typescript
import { DialogProgrammatic as Dialog } from 'buefy'

// Alert
Dialog.alert({ title: 'Notice', message: 'Operation complete.', type: 'is-info' })

// Confirm
Dialog.confirm({
  title: 'Delete item',
  message: 'Are you sure? This cannot be undone.',
  type: 'is-danger',
  hasIcon: true,
  onConfirm: () => deleteItem(),
  cancelText: 'Cancel',
  confirmText: 'Delete'
})

// Prompt
Dialog.prompt({
  message: 'Enter your name',
  inputAttrs: { placeholder: 'Name', maxlength: 50 },
  onConfirm: (value: string) => console.log(value)
})
```

---

### b-dropdown

Dropdown menu with flexible trigger and item content.

**Key props:** `v-model` (selected value), `disabled`, `inline`, `scrollable`, `max-height`, `position` (`"is-top-right"` | `"is-top-left"` | `"is-bottom-left"` | `"is-bottom-right"`), `triggers` (Array, default `['click']`), `animation`, `expanded`, `multiple`, `trap-focus`, `close-on-click`, `can-close`

**Slots:** `trigger` (scoped: `{ active }`) — toggle element; default — dropdown content (use `<b-dropdown-item>` children)

**b-dropdown-item props:** `value`, `separator`, `disabled`, `custom`, `has-link`, `paddingless`

```vue
<b-dropdown v-model="selected" aria-role="list">
  <template #trigger="{ active }">
    <b-button :label="selectedLabel" type="is-primary" :icon-right="active ? 'chevron-up' : 'chevron-down'" />
  </template>

  <b-dropdown-item value="option1" aria-role="listitem">Option 1</b-dropdown-item>
  <b-dropdown-item value="option2" aria-role="listitem">Option 2</b-dropdown-item>
  <b-dropdown-item separator />
  <b-dropdown-item value="option3" aria-role="listitem">Option 3</b-dropdown-item>
</b-dropdown>
```

---

### b-field

Wrapper for form inputs — provides label, validation state, help text, and grouping.

**Key props:** `label`, `label-for` (String — id of the input it labels), `type` (validation state: `"is-danger"` | `"is-success"` | `"is-warning"` | `"is-info"`), `message` (String | String[] — help/error text below input), `grouped`, `group-multiline`, `position` (`"is-centered"` | `"is-right"`), `expanded`, `horizontal`, `addons`, `custom-class`

**Slots:** default (the input component(s)); `label` (custom label); `message` (custom message, scoped: `{ messages }`)

```vue
<b-field label="Email" :type="errors.email ? 'is-danger' : ''" :message="errors.email">
  <b-input v-model="form.email" type="email" placeholder="user@example.com" />
</b-field>

<!-- Grouped (addons) -->
<b-field>
  <b-input expanded placeholder="Search…" />
  <p class="control">
    <b-button type="is-primary" icon-left="magnify">Search</b-button>
  </p>
</b-field>
```

---

### b-icon

Renders an icon from the registered icon pack.

**Key props:** `icon` (String — icon name, no pack prefix), `pack` (String — overrides default pack), `type`, `size`, `custom-size`, `custom-class`

```vue
<b-icon icon="home" type="is-primary" />
<b-icon icon="check-circle" size="is-large" type="is-success" />
```

---

### b-image

Lazy-loading image with aspect-ratio placeholder.

**Key props:** `src`, `alt`, `ratio` (Bulma ratio: `"1by1"` | `"4by3"` | `"16by9"` | `"square"` etc.), `placeholder`, `lazy` (Boolean, enables IntersectionObserver lazy loading), `webp-fallback`, `src-fallback`, `caption-first`

**Slots:** `placeholder` (custom placeholder), `caption`

```vue
<b-image src="/photo.jpg" alt="A photo" ratio="16by9" lazy />
```

---

### b-input

Text / textarea input.

**Key props:** `v-model` (String | Number), `type` (HTML input type, default `"text"`), `placeholder`, `size`, `icon`, `icon-right`, `icon-clickable`, `icon-right-clickable`, `loading`, `rounded`, `expanded`, `lazy`, `password-reveal`, `has-counter`, `maxlength`, `autocomplete`, `custom-class`

**Emits:** `icon-click`, `icon-right-click`, `focus`, `blur`

```vue
<b-field label="Username">
  <b-input v-model="username" placeholder="Enter username" icon="account" />
</b-field>

<b-field label="Password">
  <b-input v-model="password" type="password" password-reveal />
</b-field>

<b-field label="Bio">
  <b-input v-model="bio" type="textarea" maxlength="200" has-counter />
</b-field>
```

---

### b-loading

Full-page or container-scoped loading overlay.

**Key props:** `v-model` (Boolean — show/hide), `is-full-page` (Boolean, default `true`), `animation`, `can-cancel`

**Slots:** default (custom spinner content)

**Programmatic:**

```typescript
import { LoadingProgrammatic as Loading } from 'buefy'
const loader = Loading.open({ isFullPage: true })
// later:
loader.close()
```

```vue
<b-loading v-model="isLoading" :is-full-page="false" />
```

---

### b-menu

Vertical navigation menu (sidebar-style).

**Key props:** `accordion` (Boolean, default `true` — only one item expanded at a time), `activable`

**Slots:** default — place `<p class="menu-label">` and `<b-menu-list>` containing `<b-menu-item>` children

**b-menu-item props:** `label`, `icon`, `icon-pack`, `size`, `tag`, `active`, `expanded`, `disabled`, `animation`

**b-menu-item slots:** default (sub-items), `label` (scoped: `{ expanded, active }`)

```vue
<b-menu>
  <b-menu-list label="Navigation">
    <b-menu-item icon="home" label="Home" tag="router-link" to="/" />
    <b-menu-item icon="cog" label="Settings" :expanded="true">
      <b-menu-item label="Profile" tag="router-link" to="/settings/profile" />
      <b-menu-item label="Security" tag="router-link" to="/settings/security" />
    </b-menu-item>
  </b-menu-list>
</b-menu>
```

---

### b-message

Banner message / alert box.

**Key props:** `v-model` (Boolean — show/hide), `title`, `type`, `size`, `closable`, `has-icon`, `icon`, `icon-pack`, `auto-close`, `duration`, `progress-bar`

**Slots:** default (body content); `header` (custom header)

```vue
<b-message type="is-success" title="Success" closable>
  Your changes have been saved.
</b-message>

<b-message type="is-danger" has-icon>
  <strong>Error:</strong> Something went wrong.
</b-message>
```

---

### b-modal

Dialog overlay. Can display slot content, a Vue component, or raw HTML.

**Key props:** `v-model` (Boolean — open state), `has-modal-card` (Boolean — use `.modal-card` layout), `width` (Number|String, default `960`), `can-cancel` (Array | Boolean — `['escape', 'x', 'outside', 'button']`), `full-screen`, `trap-focus`, `animation`, `custom-class`, `custom-content-class`, `aria-role`, `aria-label`, `destroy-on-hide`

**Slots:** default (scoped: `{ close }`)

```vue
<b-button @click="isOpen = true">Open modal</b-button>

<b-modal v-model="isOpen" has-modal-card trap-focus>
  <div class="modal-card">
    <header class="modal-card-head">
      <p class="modal-card-title">My Modal</p>
    </header>
    <section class="modal-card-body">
      Modal body content.
    </section>
    <footer class="modal-card-foot">
      <b-button @click="isOpen = false">Close</b-button>
    </footer>
  </div>
</b-modal>
```

---

### b-navbar

Top navigation bar.

**Key props:** `v-model` (Boolean — mobile menu open state), `type`, `transparent`, `fixed-top`, `fixed-bottom`, `centered`, `shadow`, `spaced`, `mobile-burger`, `close-on-click`, `wrapper-class`

**Slots:** `brand` (logo area), `start` (left nav items), `end` (right nav items), `burger` (custom burger button)

**b-navbar-item props:** `tag`, `active`

```vue
<b-navbar type="is-primary" shadow>
  <template #brand>
    <b-navbar-item tag="router-link" to="/">
      <img src="/logo.svg" alt="Logo">
    </b-navbar-item>
  </template>

  <template #start>
    <b-navbar-item tag="router-link" to="/">Home</b-navbar-item>
    <b-navbar-item tag="router-link" to="/about">About</b-navbar-item>
  </template>

  <template #end>
    <b-navbar-item tag="div">
      <b-button type="is-primary" outlined>Sign up</b-button>
    </b-navbar-item>
  </template>
</b-navbar>
```

---

### b-notification

Notification bar. Inline or programmatic.

**Key props:** `v-model` (Boolean), `type`, `size`, `closable`, `has-icon`, `icon`, `icon-pack`, `auto-close`, `duration`, `progress-bar`, `position` (programmatic: `"is-top"` | `"is-top-right"` | `"is-bottom-right"` etc.)

**Slots:** default (message content)

**Programmatic:**

```typescript
import { NotificationProgrammatic as Notification } from 'buefy'
Notification.open({ message: 'Profile updated', type: 'is-success', position: 'is-top-right' })
```

```vue
<b-notification type="is-info" has-icon closable>
  You have 3 unread messages.
</b-notification>
```

---

### b-numberinput

Numeric input with +/- controls.

**Key props:** `v-model` (Number), `min`, `max`, `step`, `editable` (Boolean — allow manual typing), `controls-position` (`"compact"`), `type`, `size`, `disabled`, `placeholder`, `expanded`, `rounded`, `icon`, `icon-pack`

```vue
<b-field label="Quantity">
  <b-numberinput v-model="qty" :min="1" :max="99" />
</b-field>
```

---

### b-pagination

Page navigation.

**Key props:** `v-model` (Number — current page), `total` (Number — total items), `per-page` (Number, default `20`), `size`, `simple`, `rounded`, `order`, `range-before`, `range-after`, `icon-pack`, `icon-prev`, `icon-next`, `aria-next-label`, `aria-previous-label`, `aria-page-label`, `aria-current-label`

**Slots:** default (scoped: `{ page }`) — custom page button; `previous`, `next`

**Emits:** `update:modelValue`, `change`

```vue
<b-pagination
  v-model="currentPage"
  :total="totalItems"
  :per-page="20"
  order="is-centered"
  rounded
/>
```

---

### b-progress

Progress bar.

**Key props:** `type`, `value` (Number — current value, omit for indeterminate), `max` (Number, default `100`), `size`, `show-value`, `rounded`

**Slots:** default (custom label text, used when `show-value` is true)

```vue
<!-- Determinate -->
<b-progress type="is-success" :value="75" :max="100" show-value />

<!-- Indeterminate -->
<b-progress type="is-primary" />
```

---

### b-radio

Radio button input.

**Key props:** `v-model`, `native-value` (value set when selected), `type`, `size`, `disabled`, `required`, `name`

**Slots:** default (label text)

```vue
<b-radio v-model="picked" native-value="vue">Vue</b-radio>
<b-radio v-model="picked" native-value="react">React</b-radio>
<b-radio v-model="picked" native-value="angular">Angular</b-radio>
```

---

### b-radio-button

Radio styled as a button (use inside a `<b-field grouped>`).

**Key props:** Same as `b-radio` plus `type`

```vue
<b-field grouped>
  <b-radio-button v-model="size" native-value="S" type="is-primary">S</b-radio-button>
  <b-radio-button v-model="size" native-value="M" type="is-primary">M</b-radio-button>
  <b-radio-button v-model="size" native-value="L" type="is-primary">L</b-radio-button>
</b-field>
```

---

### b-rate

Star (or custom icon) rating input.

**Key props:** `v-model` (Number), `max` (Number, default `5`), `icon` (String, default `"star"`), `icon-pack`, `size`, `disabled`, `spaced`, `rtl`, `show-score`, `show-text`, `custom-text`, `texts` (Array\<string\> — label per value)

**Emits:** `update:modelValue`, `change`

```vue
<b-rate v-model="rating" :max="5" />
<b-rate v-model="rating" show-score />
```

---

### b-select

Select (dropdown) input.

**Key props:** `v-model`, `placeholder`, `multiple`, `native-size`, `size`, `expanded`, `loading`, `rounded`, `icon`, `icon-pack`

**Slots:** default (native `<option>` or `<optgroup>` elements)

```vue
<b-field label="Country">
  <b-select v-model="country" placeholder="Select a country" expanded>
    <option value="us">United States</option>
    <option value="ca">Canada</option>
    <option value="gb">United Kingdom</option>
  </b-select>
</b-field>
```

---

### b-sidebar

Off-canvas sidebar panel.

**Key props:** `v-model` (Boolean — open state), `type`, `overlay`, `position` (`"fixed"` | `"absolute"` | `"static"`, default `"fixed"`), `fullheight`, `fullwidth`, `right`, `mobile`, `reduce`, `expand-on-hover`, `expand-on-hover-fixed`, `can-cancel` (Array | Boolean), `delay`

**Slots:** default (sidebar content)

**Emits:** `update:modelValue`

```vue
<b-sidebar v-model="isOpen" type="is-light" fullheight>
  <div class="p-4">
    <p class="title is-4">Menu</p>
    <b-menu>...</b-menu>
  </div>
</b-sidebar>
```

---

### b-skeleton

Placeholder loading skeleton.

**Key props:** `active` (Boolean, default `true`), `animated` (Boolean, default `true`), `width` (Number | String), `height` (Number | String), `circle` (renders as circle), `rounded`, `count` (Number — how many skeleton items), `position` (`""` | `"is-centered"` | `"is-right"`), `size`

```vue
<b-skeleton width="200px" height="16px" />
<b-skeleton :count="3" />
<b-skeleton circle width="64px" height="64px" />
```

---

### b-slider

Range slider input.

**Key props:** `v-model` (Number | Number[] — pass an array for range mode), `min` (default `0`), `max` (default `100`), `step` (default `1`), `type`, `size`, `ticks`, `tooltip`, `tooltip-type`, `rounded`, `disabled`, `lazy`, `indicator`, `custom-formatter`

**Slots:** default (place `<b-slider-tick :value="n">` for custom tick marks)

```vue
<b-slider v-model="volume" :min="0" :max="100" type="is-primary" />

<!-- Range -->
<b-slider v-model="priceRange" :min="0" :max="1000" :step="50" />
```

---

### b-snackbar

Brief action notification at screen edge. Programmatic only.

```typescript
import { SnackbarProgrammatic as Snackbar } from 'buefy'

Snackbar.open('Item deleted')

Snackbar.open({
  message: 'Item deleted',
  type: 'is-warning',
  position: 'is-bottom-right',
  actionText: 'Undo',
  onAction: () => restoreItem()
})
```

---

### b-steps

Step-by-step wizard navigation.

**Key props:** `v-model` (Number | String — active step id), `type`, `size`, `animated`, `vertical`, `position`, `icon-pack`, `has-navigation`, `icon-prev`, `icon-next`

**Slots:** default (place `<b-step-item>` children); `navigation` (scoped: `{ previous, next }`) — custom prev/next buttons

**b-step-item props:** `label`, `icon`, `icon-pack`, `visible`, `step` (step marker text), `type`, `clickable`, `header-class`

```vue
<b-steps v-model="activeStep" type="is-primary">
  <b-step-item label="Account" icon="account-circle">
    Step 1 content
  </b-step-item>
  <b-step-item label="Profile" icon="account">
    Step 2 content
  </b-step-item>
  <b-step-item label="Confirm" icon="check">
    Step 3 content
  </b-step-item>
</b-steps>
```

---

### b-switch

Toggle switch (boolean or custom value).

**Key props:** `v-model`, `type`, `passive-type` (color when unchecked), `size`, `disabled`, `name`, `required`, `rounded`, `outlined`, `left-label`, `true-value`, `false-value`, `native-value`

**Slots:** default (label text shown next to switch)

```vue
<b-switch v-model="darkMode" type="is-dark">Dark mode</b-switch>
<b-switch v-model="notifications" type="is-success" passive-type="is-light">
  Notifications
</b-switch>
```

---

### b-table

Data table with sorting, pagination, filtering, and row selection.

**Key props:** `data` (Array), `columns` (Array\<TableColumnProps\>), `striped`, `bordered`, `narrowed`, `hoverable`, `loading`, `mobile-cards` (default `true`), `checkable`, `checked-rows` / `v-model:checked-rows`, `selected` / `v-model:selected`, `paginated`, `per-page` (default `20`), `current-page` / `v-model:current-page`, `default-sort` (String | String[]), `default-sort-direction` (`"asc"` | `"desc"`), `backend-sorting`, `backend-pagination`, `backend-filtering`, `total` (for backend pagination), `detailed`, `opened-detailed` / `v-model:opened-detailed`, `draggable`, `scrollable`, `sticky-header`

**Slots:** default (place `<b-table-column>` children for column definitions); `detail` (scoped: `{ row, index }`) — expanded row detail; `empty`; `footer`; `bottom-left`; `top-left`

**b-table-column props:** `field`, `label`, `sortable`, `searchable`, `visible`, `numeric`, `centered`, `width`, `custom-sort`, `custom-search`, `sticky`, `header-class`, `cell-class`

**b-table-column slots:** default (scoped: `{ row, column, index, colindex }`) — cell renderer; `header` (scoped: `{ column, index }`)

```vue
<b-table :data="users" :loading="loading" striped hoverable paginated :per-page="10">
  <b-table-column field="name" label="Name" sortable v-slot="{ row }">
    {{ row.name }}
  </b-table-column>
  <b-table-column field="email" label="Email" v-slot="{ row }">
    {{ row.email }}
  </b-table-column>
  <b-table-column label="Actions" v-slot="{ row }">
    <b-button size="is-small" @click="edit(row)">Edit</b-button>
  </b-table-column>

  <template #empty>
    <p class="has-text-centered">No records found.</p>
  </template>
</b-table>
```

---

### b-tabs

Tabbed content panel.

**Key props:** `v-model` (Number | String — active tab), `type` (tab style: `"is-boxed"` | `"is-toggle"` | `"is-toggle-rounded"`), `size`, `expanded`, `animated`, `vertical`, `position`, `multiline`, `destroy-on-hide`

**Slots:** default (place `<b-tab-item>` children); `start`; `end`

**b-tab-item props:** `label`, `icon`, `icon-pack`, `visible`, `header-class`, `disabled`

```vue
<b-tabs v-model="activeTab" type="is-boxed">
  <b-tab-item label="Profile" icon="account">
    Profile content
  </b-tab-item>
  <b-tab-item label="Settings" icon="cog">
    Settings content
  </b-tab-item>
</b-tabs>
```

---

### b-tag

Inline tag / badge / chip.

**Key props:** `type`, `size`, `rounded`, `closable`, `close-type`, `icon`, `icon-pack`, `attached`, `ellipsis`, `tabstop`, `aria-close-label`

**Slots:** default (tag label)

**Emits:** `close` (when closable and the × is clicked)

```vue
<b-tag type="is-primary">New</b-tag>
<b-tag type="is-success" rounded>Active</b-tag>
<b-tag type="is-danger" closable @close="removeTag(tag)">{{ tag }}</b-tag>

<!-- Tag list -->
<div class="tags">
  <b-tag v-for="t in tags" :key="t" closable @close="remove(t)">{{ t }}</b-tag>
</div>
```

---

### b-taginput

Multi-tag input (comma-separated tag entry with autocomplete).

**Key props:** `v-model` (Array), `data` (Array — autocomplete source), `field` (String — key in data objects for display), `type`, `size`, `rounded`, `attached`, `maxtags`, `closable`, `disabled`, `placeholder`, `open-on-focus`, `confirm-keys` (Array of keys that confirm a tag, default `['Enter', ',']`), `allow-new`, `ellipsis`, `icon`, `icon-pack`, `has-counter`

**Slots:** default (scoped: `{ option, index }`) — custom option rendering; `selected` (scoped: `{ tags }`) — custom tag chips; `tag` (scoped: `{ tag }`) — individual tag; `header`, `footer`, `empty`

```vue
<b-field label="Tags">
  <b-taginput
    v-model="tags"
    :data="suggestions"
    field="name"
    open-on-focus
    placeholder="Add a tag"
  />
</b-field>
```

---

### b-timepicker

Time picker with select-based dropdowns.

**Key props:** `v-model` (Date), `placeholder`, `inline`, `editable`, `disabled`, `size`, `expanded`, `icon`, `icon-pack`, `rounded`, `loading`, `hour-format` (`"12"` | `"24"`), `increment-minutes`, `min-time`, `max-time`, `enable-seconds`

**Slots:** `trigger`, `header`, `footer`

```vue
<b-field label="Time">
  <b-timepicker v-model="time" placeholder="Select time" hour-format="12" />
</b-field>
```

---

### b-toast

Brief status message. Programmatic only.

```typescript
import { ToastProgrammatic as Toast } from 'buefy'

Toast.open('Saved!')

Toast.open({
  message: 'Profile updated successfully.',
  type: 'is-success',
  position: 'is-top',  // 'is-top' | 'is-top-right' | 'is-top-left' | 'is-bottom' | 'is-bottom-right' | 'is-bottom-left'
  duration: 3000,
  pauseOnHover: true
})
```

---

### b-tooltip

Hover/focus tooltip.

**Key props:** `label` (String — tooltip text, or use `content` slot), `type`, `position` (`"is-auto"` | `"is-top"` | `"is-bottom"` | `"is-left"` | `"is-right"`), `triggers` (Array, default `['hover']`), `active`, `always`, `animated`, `animation`, `multilined`, `square`, `dashed`, `delay`, `close-delay`, `append-to-body`, `close-on-click`

**Slots:** default (the element that triggers the tooltip); `content` (rich tooltip content, used when `label` is not enough)

```vue
<b-tooltip label="Click to copy" type="is-dark" position="is-top">
  <b-button @click="copy">Copy</b-button>
</b-tooltip>

<!-- Rich content -->
<b-tooltip>
  <b-button>Hover me</b-button>
  <template #content>
    <strong>Rich</strong> tooltip <em>content</em>
  </template>
</b-tooltip>
```

---

### b-upload

File upload trigger (wraps a hidden `<input type="file">`).

**Key props:** `v-model` (File | File[]), `multiple`, `accept` (String — MIME types / extensions), `drag-drop`, `type`, `disabled`, `expanded`, `rounded`, `native`

**Slots:** default (the visible trigger content — button, text, drag-drop zone, etc.)

**Emits:** `update:modelValue`

```vue
<b-field>
  <b-upload v-model="files" multiple drag-drop>
    <section class="section">
      <div class="content has-text-centered">
        <b-icon icon="upload" size="is-large" />
        <p>Drop files here or click to upload</p>
      </div>
    </section>
  </b-upload>
</b-field>

<!-- Button-style trigger -->
<b-upload v-model="file" accept=".pdf,.docx">
  <b-button icon-left="upload">Choose file</b-button>
</b-upload>
```

---

## Common patterns

### Login form

```vue
<form @submit.prevent="submit">
  <b-field label="Email" :type="errors.email ? 'is-danger' : ''" :message="errors.email">
    <b-input v-model="form.email" type="email" icon="email" placeholder="user@example.com" />
  </b-field>

  <b-field label="Password" :type="errors.password ? 'is-danger' : ''" :message="errors.password">
    <b-input v-model="form.password" type="password" password-reveal icon="lock" />
  </b-field>

  <b-field>
    <b-button native-type="submit" type="is-primary" expanded :loading="loading">
      Sign in
    </b-button>
  </b-field>
</form>
```

### Data table with server-side pagination

```vue
<b-table
  :data="rows"
  :loading="loading"
  backend-pagination
  :total="total"
  :per-page="perPage"
  v-model:current-page="page"
  @page-change="fetchPage"
  backend-sorting
  @sort="onSort"
  striped
  hoverable
>
  <b-table-column field="id" label="ID" sortable numeric v-slot="{ row }">
    {{ row.id }}
  </b-table-column>
  <b-table-column field="name" label="Name" sortable v-slot="{ row }">
    {{ row.name }}
  </b-table-column>
</b-table>
```

### Confirmation before delete

```typescript
import { DialogProgrammatic as Dialog } from 'buefy'

function confirmDelete(id: number) {
  Dialog.confirm({
    title: 'Delete record',
    message: 'This action <strong>cannot be undone</strong>.',
    type: 'is-danger',
    hasIcon: true,
    onConfirm: () => deleteRecord(id)
  })
}
```

---
> Source: [buefy/buefy](https://github.com/buefy/buefy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
