## arrive

> A lightweight JavaScript library (v2.5.2) for watching DOM element creation and removal using Mutation Observers.

# arrive.js

A lightweight JavaScript library (v2.5.2) for watching DOM element creation and removal using Mutation Observers.

## Project Structure

```
src/arrive.js          # Main source file
minified/arrive.min.js # Minified build
tests/
  spec/arriveSpec.js   # Jasmine test specs
  lib/jasmine-2.0.0/   # Jasmine test framework
  SpecRunner.html      # Test runner (open in browser)
  withoutjQuery.js     # Tests without jQuery
```

## Running Tests

Tests use Jasmine 2.0 and run in-browser:

```bash
npm test   # Opens tests/SpecRunner.html in browser
```

There is no CLI test runner — open `tests/SpecRunner.html` directly in a browser.

## Building

There is no build script. To update the minified file, manually minify `src/arrive.js` into `minified/arrive.min.js`.

## Key Architecture

- `src/arrive.js` wraps `MutationObserver` to provide `arrive` (creation) and `leave` (removal) events
- Works with `document`, `HTMLElement`, `HTMLDocument`, `Window`, `HTMLCollection`, and `NodeList`
- Optional jQuery integration — attaches to `$.fn` when jQuery is present, but has zero required dependencies
- `Arrive` is the global export; also exposes `Arrive.unbindAllArrive()` and `Arrive.unbindAllLeave()`

## API

```javascript
// Watch for element creation
element.arrive(selector, [options], callback)
element.arrive(selector)  // Promise-based (resolves once)

// Watch for element removal
element.leave(selector, [options], callback)

// Unbind
element.unbindArrive([selector], [callback])
element.unbindLeave([selector], [callback])
Arrive.unbindAllArrive()
Arrive.unbindAllLeave()
```

### Options

| Option | Default | Description |
|--------|---------|-------------|
| `fireOnAttributesModification` | `false` | Watch attribute changes on existing elements |
| `onceOnly` | `false` | Fire callback once, then auto-unbind |
| `existing` | `false` | Include elements already in the DOM |
| `timeout` | `0` | Auto-unbind after N ms, call callback with `null` (0 = disabled) |

---
> Source: [uzairfarooq/arrive](https://github.com/uzairfarooq/arrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
