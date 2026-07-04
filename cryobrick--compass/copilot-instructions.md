## compass

> handleSelect: function(action, index) {},

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Compass is an air-gapped Bitcoin wallet for KaiOS 2.5 feature phones (Firefox 48). It uses pure vanilla JavaScript (ES2015) with no frameworks. The app disguises itself as "Offline Maps & Bookmarks" for plausible deniability.

**Target:** KaiOS 2.5, Firefox 48, 240x320px screen, D-pad navigation, ES2015 only (no async/await).

## Build & Development Commands

```bash
# Bundle dependencies (run after npm install of respective packages)
./bundle-bip39.sh      # BIP39 mnemonic library
./bundle-address.sh    # BIP32 + bitcoinjs-lib for address generation
./bundle-qrcode.sh     # QR code generator

# Build OmniSD package for KaiOS
./build.sh             # Creates dist/compass-v{VERSION}.zip

# Browser testing
open index.html        # ↑↓ navigate, Enter select, Esc back, Q/E soft keys

# KaiOS simulator
python3 -m http.server 8080
# Then open http://localhost:8080/simulator.html
```

There is no test framework, linter, or package.json in the project root. Testing is manual via browser or KaiOS device/simulator.

## Architecture

### Screen-based SPA Pattern

`js/app.js` manages a screen registry. Each screen object follows this interface:

```javascript
var ScreenName = {
  id: "screen-id",          // matches HTML element id
  render: function() {},     // returns HTML string
  onEnter: function() {},    // setup navigation callbacks
  onExit: function() {},     // cleanup
  handleSelect: function(action, index) {},
  handleBack: function() {}
};
```

`js/navigation.js` handles all D-pad/keyboard input and focus management (`.focusable`/`.menu-item` elements, `.focused` CSS class).

### Service Layer (IIFE modules)

- **`js/services/wallet.js`** — BIP39 mnemonic generation, BIP32/BIP84 key derivation, Native SegWit (P2WPKH bc1...) address generation, zpub export via custom SLIP-132 network params
- **`js/services/pin.js`** — 6-digit PIN with SHA-256(salt + PIN) hashing via `crypto.subtle`
- **`js/services/psbt-alternative.js`** — Custom minimal PSBT parser/signer using only bitcoinjs-lib (no external PSBT library). BIP143 signing, outputs transaction hex.

### Library Bundles (generated, in `js/lib/`)

Pre-bundled with esbuild targeting ES2015. These are generated (gitignored) artifacts:
- `bip39-bundle.js` — bip39@2.5.0
- `address-bundle.js` — bip32@1.0.2 + bitcoinjs-lib@3.3.2 + crypto-browserify
- `qrcode-bundle.js` — qrcode-generator@2.0.4

Library versions are pinned for Firefox 48 compatibility. Do not upgrade without testing on KaiOS.

### Storage

All data in `localStorage`, unencrypted:
- `compass_wallet` — `{mnemonic, createdAt}`
- `compass_pin_salt` / `compass_pin_hash` — PIN verification data

### Key Derivation Path

BIP84 Native SegWit: `m/84'/0'/0'/{change}/{index}` where change=0 (receiving) or 1 (change).

### App Flow

Splash → CreatePIN (first time) → Compass (plausible deniability) → PIN unlock → Welcome (create/restore wallet) → Home (sign PSBT, explore addresses, export zpub, logout).

## Platform Constraints

- No arrow functions in some contexts (Firefox 48 edge cases) — use `function()` when unsure
- Polyfills included: Buffer, process (process-shim.js), crypto-browserify, stream-browserify
- Screen is 240x320px — all UI must fit this constraint
- Navigation is D-pad only (ArrowUp/Down, Enter, Escape, SoftLeft/SoftRight mapped to Q/E)

---
> Source: [cryobrick/compass](https://github.com/cryobrick/compass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
