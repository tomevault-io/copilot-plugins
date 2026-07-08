## playground-showcase

> Use this task whenever you need to add a new website to the Playground Showcase.

# Claude Task: Add a New Site to the Showcase

Use this task whenever you need to add a new website to the Playground Showcase.

---

## Inputs Required

Before starting, collect the following from the user (or accept them inline):

| Field | Example |
|---|---|
| Site URL | `https://mysite.com` |
| Display name | `MySite` |
| One-sentence description | `A tool that does X.` |
| Device type | `phone`, `laptop`, or `tablet` |

If the user doesn't specify a device type, choose based on the site:
- **phone** — mobile-first apps, simple tools, single-page experiences
- **laptop** — dashboards, data-heavy sites, marketing pages
- **tablet** — content-rich apps, editorial layouts, portfolio pieces

---

## Step 1 — Capture a Screenshot

Use your browser functionality to navigate to the site URL and take a full-page screenshot.

- Save it to `./assets/screen-<slug>.jpg`
  - `<slug>` = lowercase site name, no spaces or special characters (e.g. `mysite`, `tight-lines`)
- The image should capture the full page height (not just the viewport) so the scroll animation has content to travel through
- If the site is password-protected or unavailable, ask the user to provide a screenshot file

---

## Step 2 — Read the Current Layout

Read `index.html` to:
1. Count the existing devices (numbered comments `<!-- Device N: ... -->`)
2. Note the `--x`, `--y`, and `--rot` values already in use

**Existing device positions (for reference):**
- Row 1 (`--y: ~2–8%`): x=3%, x=15%, x=43%
- Row 2 (`--y: ~50–55%`): x=8%, x=20%, x=47%

Pick a position that fills a visible gap, or start a new row at `--y: 100%` if both rows are full. Alternate the rotation sign from neighbouring devices (±2–8deg).

---

## Step 3 — Insert the Device Block

Edit `index.html` to insert a new device block **before the closing `</div>`** of `<div class="device-floor" id="deviceFloor">`.

Use this template, filling in every `[placeholder]`:

```html
    <!-- Device [N]: [DeviceType] — [Site Name] -->
    <a href="[URL]" target="_blank" rel="noopener noreferrer"
       class="device device--[phone|laptop|tablet]"
       style="--x: [X]%; --y: [Y]%; --rot: [degrees]deg;"
       data-scroll-speed="[speed]">
      <div class="device__screen-clip device__screen-clip--[phone|laptop|tablet]">
        <img class="device__screenshot" src="./assets/screen-[slug].jpg" alt="" draggable="false">
      </div>
      <img class="device__frame" src="./assets/frame-[iphone|macbook|ipad].png" alt="[DeviceType] showing [Site Name]" draggable="false">
      <div class="device__label">
        <span class="device__title">[Site Name]</span>
        <span class="device__desc">[One-sentence description.]</span>
      </div>
    </a>
```

**Class and asset mapping:**

| Device type | `class` | `frame src` | `screen-clip class` | `data-scroll-speed` |
|---|---|---|---|---|
| phone | `device--phone` | `frame-iphone.png` | `device__screen-clip--phone` | `0.12`–`0.15` |
| laptop | `device--laptop` | `frame-macbook.png` | `device__screen-clip--laptop` | `0.08` |
| tablet | `device--tablet` | `frame-ipad.png` | `device__screen-clip--tablet` | `0.10` |

> **No changes needed** to `style.css` or `app.js` — they automatically pick up any new `.device` element.

---

## Step 4 — Verify

1. Read the modified `index.html` and confirm:
   - The new block is syntactically correct
   - The device number in the comment is sequential
   - The screenshot `src` path matches the saved file name exactly
2. Confirm the screenshot file exists at `./assets/screen-<slug>.jpg`

---

## Step 5 — Commit and Push

Stage and commit with a descriptive message:

```
Add [Site Name] to showcase ([device type] device)
```

Push to the current working branch.

---

## Device Frame Reference

The three device frame PNGs are in `./assets/`:

| File | Dimensions | Screen area (% of frame) |
|---|---|---|
| `frame-iphone.png` | 400 × 788 px | left: 8.50%, top: 4.31%, w: 83%, h: 91.37% |
| `frame-macbook.png` | 800 × 520 px | left: 11.75%, top: 11.73%, w: 76.50%, h: 76.54% |
| `frame-ipad.png` | 500 × 690 px | left: 6%, top: 4.35%, w: 88%, h: 91.30% |

Screenshots are clipped to these areas by CSS — the frame image sits on top with `z-index: 2`, and the screenshot scrolls within the clip container underneath.

---
> Source: [jordanbmcgowen/Playground-showcase](https://github.com/jordanbmcgowen/Playground-showcase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
