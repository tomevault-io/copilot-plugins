## rightscreen

> Web app for generating smartphone lockscreen images that assert 4th Amendment rights. Users select device, background, text message, and font, then download a PNG.

# RightScreen

## Project Overview
Web app for generating smartphone lockscreen images that assert 4th Amendment rights. Users select device, background, text message, and font, then download a PNG.

## Tech Stack
- Vanilla HTML/CSS/JS (no frameworks, no build tools)
- Canvas API for image rendering
- Static files - can be served from any web server

## File Structure
```
index.html          # Single page app entry point
css/styles.css      # All styling (dark theme, mobile-first)
js/
  devices.js        # Device definitions with dimensions and safe zones
  config.js         # Colors, gradients, text options, fonts
  canvas.js         # LockscreenRenderer class for canvas operations
  app.js            # RightScreenApp class - UI logic and state
```

## Key Patterns

### Adding a new device
Edit `js/devices.js` - add to appropriate manufacturer group:
```js
{ id: 'device-id', name: 'Display Name', width: 1080, height: 2400, safeZoneTop: 80, safeZoneBottom: 0 }
```

### Adding a new background color
Edit `js/config.js` COLORS array:
```js
{ id: 'color-id', name: 'Display Name', value: '#hexcode', textColor: '#ffffff' }
```

### Adding a new gradient
Edit `js/config.js` GRADIENTS array - requires both CSS string and canvas stops.

### Adding a new text option
Edit `js/config.js` TEXT_OPTIONS array:
```js
{ id: 'text-id', text: 'The message text' }
```

## Development
```bash
python3 -m http.server 8080
```
Open http://localhost:8080

## Notes
- Safe zones prevent text from overlapping camera cutouts (notch, dynamic island, punch-hole)
- Preview uses sticky positioning to remain visible while scrolling
- State persists to localStorage
- Share Image uses Web Share API on mobile (falls back to download + prompt on desktop)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johnzilla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
