## getbudgeter-com

> This is a web app for users to manage their monthly bills and track if they've been paid.

# Budgeter

This is a web app for users to manage their monthly bills and track if they've been paid.

## Tech Stack

- Django
- HTMX
- Web Awesome (formerly Shoelace)

## Code style

The frontend uses as little client-side JavaScript as possible. Default to HTMX for any user
interaction that involves data. Create new endpoints liberally rather than trying to make JS do
the work.

When adding client-side JavaScript for non-HTTP interactions (e.g., DOM manipulation, event handling,
UI effects), first consider using [surrealjs](https://github.com/gnat/surreal) if appropriate.

When writing python code, prefer top-level imports as opposed to imports inside function bodies.

---

## HTMX Patterns

### Opening drawers/dialogs
Always append to `<body>` so Web Awesome can teleport the overlay correctly:
```html
<wa-button hx-get="/things/new" hx-target="body" hx-swap="beforeend">Add</wa-button>
```

### Fragment targets
Define named fragments with `{% partialdef %}` and target them by ID:
```html
{% partialdef thing-list inline %}
  <div id="thing-list">...</div>
{% endpartialdef thing-list %}
```
The view returns the fragment via `render(request, "things/page.html#thing-list", context)`.

### Returning updated state after mutations (POST/PUT/DELETE)
After a mutation, re-render the relevant list fragment and set `HX-Trigger` so sidebars/stats
reload without a full page refresh:
```python
response = render(request, "things/page.html#thing-list", context)
response["HX-Trigger"] = "things-changed"
return response
```

### Listening for cross-component events
Sidebar panels load on page load and re-load when data changes:
```html
<div hx-get="/things/stats" hx-trigger="load, things-changed from:body"></div>
```

### Month-scoped pages
Pages that show monthly data use a `wa-input type="month"` with HTMX. The input fires a GET
that pushes the new URL, so the month is bookmarkable:
```html
<wa-input
  type="month"
  id="month"
  value="{{ selected_month|date:'Y-m' }}"
  hx-get="/things"
  hx-push-url="true"
  hx-target="#things-list"
  hx-trigger="change delay:500ms"
  hx-swap="outerHTML"
></wa-input>
```
The view reads the month from `request.GET.get("month")`, defaulting to the current month.
Month navigation buttons call a small JS helper (`changeMonth(+/-1)`) that updates the input value
and fires `change`; this is the only JS needed.

### Passing context to HTMX GET requests
When a button needs to carry a value (e.g., current month) into a GET endpoint, use a query param:
```html
<wa-button hx-get="/things/new?month={{ selected_month|date:'Y-m' }}" hx-target="body" hx-swap="beforeend">
```

### Delete with confirmation
Use `customConfirm()` (defined in `static/js/app.js`) instead of the native `hx-confirm`:
```html
<wa-button
  hx-delete="/things/{{ thing.id }}"
  hx-trigger="confirmed"
  hx-target="#thing-list"
  hx-swap="outerHTML"
  onclick="customConfirm().then(yes => { if (yes) htmx.trigger(this, 'confirmed') })"
>
```

### Forms inside drawers
- Put the form *inside* `<wa-drawer>` with a stable `id`
- Put Cancel/Save in `slot="footer"` with a 2-column grid layout
- Cancel: `onclick="this.closest('wa-drawer').open = false"`
- Save: `type="submit" form="<form-id>"`
- `app.js` already handles auto-closing the drawer on successful HTMX response

```html
<wa-drawer label="Add Thing">
  <form id="thing-form" hx-post="/things" hx-target="#thing-list" hx-swap="outerHTML"
        style="display: flex; flex-direction: column; gap: 1rem;">
    {% csrf_token %}
    ...
  </form>
  <div slot="footer" style="width: 100%; display: grid; grid-template-columns: 1fr 1fr; gap: 1rem;">
    <wa-button appearance="outlined" onclick="this.closest('wa-drawer').open = false"
               style="margin-top: 1rem; width: 100%;">Cancel</wa-button>
    <wa-button type="submit" form="thing-form" style="margin-top: 1rem; width: 100%;">Save</wa-button>
  </div>
</wa-drawer>
```

---

## Web Awesome Components

Use `wa-*` components consistently. Key components and their usage:

| Component | Usage |
|-----------|-------|
| `wa-button` | All buttons. Use `appearance="plain"` for icon-only, `appearance="outlined"` for secondary, `variant="danger"` for destructive |
| `wa-icon` | Icons via Font Awesome name. Use `slot="start"` inside buttons for leading icon |
| `wa-input` | Text, number, url, month inputs. Always use `label=` instead of external `<label>` |
| `wa-select` / `wa-option` | Dropdowns in forms. Use `value=` on `wa-select` to set the selected option |
| `wa-checkbox` | Boolean toggles in forms |
| `wa-drawer` | Side-panel forms (add/edit). Preferred over `wa-dialog` for forms |
| `wa-dialog` | Modal dialogs for confirmations or short non-form interactions |
| `wa-dropdown` / `wa-dropdown-item` | Action menus (e.g., page-level ⋮ menu) |
| `wa-badge` | Status chips |

---

## Surrealjs

Use surrealjs for any client-side DOM work that doesn't need a server round-trip.

### Key rules
- `me()` without args = parent element of the `<script>` tag. Works reliably for non-web-component
  parents. Use `me('#id')` (explicit selector) when the script might run after an HTMX swap.
- `any(selector)` = all matching elements, chainable.
- **Do NOT use space-separated events**: `me().on("event1 event2", fn)` does NOT work.
  Use separate `.on()` calls.
- Avoid `me()` inside `<script>` children of web components — the context is unreliable after
  HTMX swaps. Use document-level event delegation instead.

### Examples

```html
<!-- Event handling with me() - gets the parent of the script tag -->
<button>
  <script>
    me().on("click", () => me().classToggle("active"))
  </script>
  Toggle
</button>

<!-- Selecting and manipulating multiple elements with any() -->
<script>
  any(".collapsible").on("click", e => {
    me(e).classToggle("expanded")
  })
</script>

<!-- Fade effects and animations -->
<script>
  me("#notification").fadeOut() // Fade out then remove
  me("#modal").fadeIn()         // Fade in element with opacity: 0
</script>

<!-- Class and style manipulation -->
<script>
  me(".card").classAdd("highlighted")
  me(".card").styles("border: 2px solid blue")
  me(".card").styles({ backgroundColor: "yellow", padding: "10px" })
</script>

<!-- Toggle data attributes (used for CSS-driven show/hide) -->
<div id="container">
  <script>
    me().on("click", () => {
      const el = me('#container')
      el.attr('data-edit-mode', el.attr('data-edit-mode') === 'true' ? null : 'true')
    })
  </script>
</div>
```

---

## Template Partials

Use `django-template-partials` (`{% partialdef %}`) to co-locate fragments with their parent
template. HTMX can target a named fragment with `template.html#fragment-name`.

### Pattern
```html
{% partialdef thing-list inline %}
  <div id="thing-list">
    {% for thing in things %}
      {% partialdef thing-item inline %}
        <li id="thing-{{ thing.id }}">{{ thing.name }}</li>
      {% endpartialdef thing-item %}
    {% endfor %}
  </div>
{% endpartialdef thing-list %}
```

### Views
```python
# Full page load
return render(request, "things/page.html", context)

# HTMX partial swap
return render(request, "things/page.html#thing-list", context)

# Detect HTMX vs full load
if request.htmx and not request.htmx.boosted:
    template = "things/page.html#thing-list"
else:
    template = "things/page.html"
```

---

## Notes

- We have a custom build of Web Awesome. The `./frontend` dir is just for cherry-picking WA
  components and creating a frontend build to copy into Django static dirs.
- `app.js` defines HTMX extensions loaded on `<body>`:
  - `csrf` — injects CSRF token into HTMX requests
  - `button-spinner` — disables submit buttons and shows spinner during request
  - `show-hide-dialogs` — auto-opens drawers/dialogs appended to body; auto-removes on close
  - `head-support` — allows `<head>` tags in HTMX responses (used for CSS injection on partial loads)
- `customConfirm()` (in `app.js`) is a styled confirmation dialog. Use it instead of
  `hx-confirm` for consistency with the rest of the UI.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mulleady1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
