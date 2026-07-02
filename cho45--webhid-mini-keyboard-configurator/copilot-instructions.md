## webhid-mini-keyboard-configurator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a WebHID-based keyboard configurator for MINI keyboards (vendor ID: 0x1189, product ID: 0x8890). It provides a web interface to configure key mappings, media keys, mouse buttons, and LED patterns on the device.

## Architecture

The project consists of two main files:
- **index.html**: Vue.js-based UI with internationalization (Japanese/English)
- **KeyConfigurator.js**: HID communication protocol implementation

Key architectural components:
1. **WebHID API**: Used for USB device communication
2. **Vue 3 + Vue-i18n**: UI framework with internationalization
3. **HID Protocol**: Custom protocol for keyboard configuration
   - Report IDs: 3, 0, 2 (tried in order)
   - Command structure: 8-byte packets
   - Key types: 0x01 (keyboard), 0x02 (media), 0x03 (mouse)
   - Special commands: 0xA1 (layer), 0xAA (flash write), 0xB0 (LED)

## Development Commands

Since this is a static web project with no build process:

### Running the project
```bash
npx serve
```

### Testing
- Open in Chrome or Edge (WebHID API required)
- Connect a compatible keyboard device
- Test key configuration and LED control
- Verify both Japanese and English UI

## Important Implementation Details

### HID Protocol Structure
- **Single Key**: modifier (index 0) + modifier+keycode (index 1)
- **Macro/Keystroke**: Up to 6 keystrokes, modifier applies only to first keystroke
- **Layer Support**: 3 layers (currently UI only shows layer 1)
- **Flash Write**: Command 0xAA,0xAA for key configs, 0xAA,0xA1 for LED

### Key Configuration Flow
1. Connect to device and determine working Report ID
2. Set layer if needed (Report ID != 0)
3. Send configuration commands
4. Write to flash memory

### Error Handling
- Internationalized error messages using Vue-i18n
- Graceful fallback for unsupported browsers
- Connection retry with multiple Report IDs

## UI/UX Conventions

- Material Design-inspired styling
- Real-time status updates in log area
- Visual feedback for recording state
- Warning messages for device limitations (e.g., modifier keys in macros)

## Browser Compatibility

- Chrome/Edge: Full support
- Firefox/Safari: Not supported (no WebHID API)
- Must be served over HTTPS or localhost for WebHID access

---
> Source: [cho45/webhid-mini-keyboard-configurator](https://github.com/cho45/webhid-mini-keyboard-configurator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
