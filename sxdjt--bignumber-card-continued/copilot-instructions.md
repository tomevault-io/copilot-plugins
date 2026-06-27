## bignumber-card-continued

> This file provides detailed guidance for working with the bignumber-card-continued project.

# CLAUDE.md - Big Number Card Continued

This file provides detailed guidance for working with the bignumber-card-continued project.

## Project Overview

Community-maintained continuation of the original bignumber-card by [@ciotlosm](https://github.com/ciotlosm). The original project has not been updated since January 2022. This continuation aims to maintain compatibility, fix bugs, and incorporate valuable community contributions.

## Continuation Philosophy

Following the same principles as canvas-gauge-card-continued in this monorepo:

1. **Respect Original Work**: Credit original authors, maintain git history when possible
2. **Prioritize Stability**: Bug fixes and compatibility updates before new features
3. **Community-Driven**: Review and merge valuable PRs from the original repository
4. **Conservative Versioning**: Use `-continued` suffix (e.g., `0.0.7-continued` or `1.0.0-continued`)
5. **Maintain Simplicity**: Keep the single-file vanilla JS approach - no build system needed

## Architecture

### Technology Stack

- **Framework**: Vanilla JavaScript Web Components (no dependencies)
- **Build System**: None required
- **File Structure**: Single file (`bignumber-card.js`)
- **Browser Support**: Modern browsers with Web Components support

### Code Structure

**Class**: `BigNumberCard extends HTMLElement`

**Key Methods**:
- `constructor()`: Creates Shadow DOM attachment
- `setConfig(config)`: Validates config, builds DOM structure, applies initial styles
- `set hass(hass)`: Called on every Home Assistant state update, updates displayed values
- `getCardSize()`: Returns card height hint (always returns 1)

**Helper Methods**:
- `_fire(type, detail, options)`: Dispatches custom events (used for more-info dialog)
- `_computeSeverity(stateValue, sections)`: Finds matching severity level
- `_getColor(entityState, config)`: Resolves text color (severity → config → default)
- `_getStyle(entityState, config)`: Resolves background color (severity → config → default)
- `_translatePercent(value, min, max)`: Converts value to progress bar percentage

### Rendering Approach

**Shadow DOM with Direct Manipulation**:
1. `setConfig()` builds DOM structure once:
   - Creates `<ha-card>` container
   - Adds `#value` div for the number display
   - Adds `#title` div for the title
   - Injects `<style>` tag with CSS custom properties
2. `set hass()` updates content via direct DOM manipulation:
   - Modifies CSS custom properties for dynamic styling
   - Updates text content of value/title elements
   - Applies/removes CSS classes for None state handling

**Why This Approach**:
- Lightweight: ~150 lines, no dependencies
- Fast: No virtual DOM overhead for such simple updates
- Compatible: Uses standard Web Components APIs

### CSS Architecture

**CSS Custom Properties** (set dynamically):
- `--bignumber-color`: Text color
- `--bignumber-fill-color`: Background/bar color
- `--bignumber-percent`: Progress bar fill percentage (100% = empty, 0% = full)
- `--bignumber-direction`: Bar fill direction (left/right/top/bottom)
- `--base-unit`: Base scale for all sizing (default: 50px)

**Gradient Background**:
Uses CSS linear-gradient to create progress bar effect:
```css
background: linear-gradient(
  to var(--bignumber-direction),
  var(--card-background-color) var(--bignumber-percent),
  var(--bignumber-fill-color) var(--bignumber-percent)
);
```

## Configuration System

### Required Options
- `entity`: Sensor entity ID (e.g., `sensor.temperature`)

### Display Options
- `title`: Card title text
- `scale`: Base unit for sizing (default: "50px")
- `attribute`: Entity attribute to display instead of state
- `hideunit`: Boolean to hide unit of measurement
- `round`: Number of decimal places for rounding

### Styling Options
- `color`: Text color (hex or CSS variable)
- `bnStyle`: Background/bar color (hex or CSS variable)
- `opacity`: Opacity for unit text (default: "0.5")

### Progress Bar Options
- `min`: Minimum value (enables bar display)
- `max`: Maximum value (required if min is set)
- `from`: Fill direction - "left", "right", "top", or "bottom" (default: "left")

### Severity System
Array of objects defining value-based color changes:
```javascript
severity: [
  { value: 70, bnStyle: 'var(--label-badge-green)' },
  { value: 90, bnStyle: 'var(--label-badge-yellow)', color: '#000' },
  { value: 100, bnStyle: 'var(--label-badge-red)', color: '#FFF' }
]
```

**Rules**:
- Must be in ascending order by `value`
- Values represent upper thresholds
- First matching threshold is used
- Can override both `bnStyle` and `color`

### None/Offline State Handling
Special configuration for handling None/unavailable sensors:
- `noneString`: Display text when value is None (e.g., "Offline")
- `noneCardClass`: CSS class to add to `<ha-card>` when None
- `noneValueClass`: CSS class to add to `#value` when None

## Known Issues

### Bug in Original Code
**Line 21**: Typo in property name
```javascript
if (!cardConfig.noneString) cardConfig.nonestring = null;
```
Should be `noneString` (capital S) for consistency. This bug doesn't break functionality but indicates code quality issues.

### Missing Error Handling
**Line 115**: No validation before accessing entity
```javascript
const entityState = config.attribute
  ? hass.states[config.entity].attributes[config.attribute]
  : hass.states[config.entity].state;
```
Will throw error if `hass.states[config.entity]` is undefined.

### Other Issues from Original Repo
- Colors disappearing on Amazon Fire Tablets (Issue #43)
- Loading issues on some platforms (Issues #8, #16)
- iOS/iPad rendering bugs when max value exceeded (Issue #7)

## Pending Community Features

### Open Pull Requests (from original repo)

1. **PR #48: Configurable Tap Actions**
   - Replaces hardcoded more-info with standard HA tap actions
   - Adds support for: navigation, URL, service calls, toggle
   - Backwards compatible (defaults to more-info)

2. **PR #47: Font Size Customization**
   - New options: `title_font_size`, `value_font_size`, `card_padding`
   - Decouples card size from font size
   - Fixes text overflow issues

3. **PR #46: Locale-Aware Formatting**
   - Uses `toLocaleString()` for number formatting
   - Automatic thousands separators based on browser locale
   - Example: 19578 → "19,578" (US) or "19.578" (DE)

### Requested Features (from issues)
- Background color control (Issue #34)
- Additional font customization options
- More flexible positioning and layout options

## Development Workflow

### Testing Changes

**No Build Required** - Edit directly and copy:

1. Edit `bignumber-card.js`
2. Copy to Home Assistant:
   ```bash
   cp bignumber-card.js /path/to/homeassistant/config/www/
   ```
3. Hard refresh browser (Cmd+Shift+R / Ctrl+Shift+F5)
4. Check browser console for errors (F12)

### Adding Configuration Options

Standard pattern:

1. **Add default in `setConfig()`**:
   ```javascript
   if (!cardConfig.myOption) cardConfig.myOption = 'default';
   ```

2. **Use in rendering**:
   ```javascript
   const value = config.myOption;
   ```

3. **Document in README.md** under Configuration Options table

4. **Update CHANGELOG.md** with feature description

### Modifying Styles

**To change default colors**:
- Edit `_DEFAULT_STYLE()` for background color
- Edit `_DEFAULT_COLOR()` for text color

**To add style options**:
1. Add configuration property in `setConfig()`
2. Apply via CSS custom property in `set hass()`
3. Document in README

### Severity System Modifications

**Current logic** (`_computeSeverity`):
- Iterates through severity array
- Returns first item where `stateValue <= section.value`
- Returns undefined if no match

**To modify**:
- Consider descending order support
- Add validation for ascending values
- Better error messages for misconfigurations

## Version Management

**Current Status**: Unreleased continuation

**Suggested First Release**: `0.0.7-continued` or `1.0.0-continued`

**Release Process**:
1. Update version in `hacs.json`
2. Document changes in `CHANGELOG.md`
3. Test thoroughly in Home Assistant
4. Create git tag matching version
5. Create GitHub release with tag
6. No build artifacts needed (single JS file)

## Security Considerations

### Input Sanitization
Currently minimal - relies on Home Assistant for entity validation.

**Areas for improvement**:
- Validate entity IDs before accessing `hass.states`
- Sanitize configuration strings used in CSS
- Validate numeric inputs (min, max, scale)

### XSS Risks
**Current exposure**:
- `innerHTML` usage in line 129 for unit display
- User-provided strings in CSS custom properties

**Mitigation**:
- Replace `innerHTML` with `textContent` where possible
- Sanitize or validate user-provided CSS values
- Consider CSP-compatible styling approaches

## Testing Strategy

### Manual Testing Checklist
- [ ] Basic numeric sensor display
- [ ] Severity level color changes
- [ ] Progress bar with min/max
- [ ] All four fill directions (left/right/top/bottom)
- [ ] Entity attribute display
- [ ] Unit hiding (`hideunit: true`)
- [ ] Decimal rounding (`round: 2`)
- [ ] None state handling (offline sensor)
- [ ] Custom colors (hex and CSS variables)
- [ ] Title display
- [ ] Scale sizing
- [ ] More-info dialog on click

### Edge Cases
- Entity doesn't exist
- Entity state is unavailable/unknown
- Non-numeric state values
- Very large/small numbers
- Negative numbers with progress bar
- Missing required configuration
- Invalid severity configuration

## Code Quality Improvements

### Quick Wins
- [ ] Fix `nonestring` → `noneString` typo
- [ ] Add error handling for missing entities
- [ ] Add JSDoc comments
- [ ] Extract magic numbers to constants
- [ ] Validate severity array ordering

### Larger Refactors (Consider Carefully)
- Template-based rendering (may complicate simplicity)
- TypeScript conversion (would require build system)
- Split into multiple files (goes against single-file philosophy)

## Contributing Guidelines

### PRs to Review First
1. Existing PRs from original repo (#46, #47, #48)
2. Bug fixes for known issues
3. Compatibility updates for Home Assistant changes

### New Feature Criteria
- Maintains single-file simplicity
- No external dependencies
- Backwards compatible with existing configs
- Clear use case with user demand
- Doesn't over-complicate the card's purpose

### Code Style
- Match existing vanilla JS style
- Use ES6+ features where appropriate
- Keep functions small and focused
- Prioritize readability over cleverness

## Resources

- Original Repository: https://github.com/custom-cards/bignumber-card
- Home Assistant Developer Docs: https://developers.home-assistant.io/
- Web Components Docs: https://developer.mozilla.org/en-US/docs/Web/Web_Components
- HACS Documentation: https://hacs.xyz/

---
> Source: [sxdjt/bignumber-card-continued](https://github.com/sxdjt/bignumber-card-continued) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
