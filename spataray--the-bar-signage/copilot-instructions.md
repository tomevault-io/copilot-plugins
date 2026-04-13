## the-bar-signage

> T.A. Station Digital Signage is a real-time, web-based display system for a Thai hostess bar in Honolulu, HI. The system consists of two primary interfaces:

# GEMINI.md

## Project Overview
T.A. Station Digital Signage is a real-time, web-based display system for a Thai hostess bar in Honolulu, HI. The system consists of two primary interfaces:
- **TV Display (`index.html`)**: A fullscreen dashboard that plays YouTube videos/music, rotates advertisements, displays a drinks menu, and shows a scrolling ticker with weather, time, and fun facts.
- **Staff Remote (`maintenance.html`)**: A mobile-optimized control panel used by staff to manage the queue, search YouTube, toggle "Karaoke Mode", trigger ads, and switch themes.

## Architecture & Data Flow
- **Real-time Sync**: Uses **Firebase Realtime Database** to synchronize state between the remote and the TV instantly.
- **Content Management**:
  - **Dynamic Data**: Ads, menu items, and ticker messages are fetched from **Google Sheets** (published as CSV).
  - **Fun Facts**: Periodically updated via a Python script (`update_facts.py`) using **Anthropic's Claude API**, which injects new content directly into `index.html`.
- **Media Playback**: Integrates the **YouTube IFrame API** for video playback and the **YouTube Data API v3** for searching videos on the remote.
- **Hosting**: Deployed via **GitHub Pages**.

## Key Files
- `index.html`: The main TV signage interface. Contains the YouTube player, sidebar, and ticker.
- `maintenance.html`: The staff remote control interface.
- `style.css`: Styles for the TV display, including "Party Mode" themes and layout.
- `update_facts.py`: Python automation script to generate Thai/Hawaii fun facts.
- `.github/workflows/update-facts.yml`: GitHub Action to run the fact-updating script on a schedule.
- `images/`: Directory containing bar photos used for the idle slideshow and sidebar background.

## Development Conventions
- **Legacy Compatibility**: Uses the **Firebase JS SDK v8** (namespaced) and standard `var` declarations in JavaScript. Maintain this style for consistency.
- **No Build System**: This is a pure static project. There are no bundlers (Webpack/Vite) or package managers (NPM/Yarn). JS and CSS are mostly inline or linked directly.
- **Direct Injection**: The `update_facts.py` script uses regex to replace content between `// AUTO-GENERATED-FACTS-START` and `// AUTO-GENERATED-FACTS-END` markers in `index.html`.
- **CSS Transitions**: The ad sidebar visibility is controlled via a CSS grid transition on the `--sidebar-width` variable.
- **Version Tracking**: Always update the `v1.x.x (Date Time)` version tag at the bottom of `index.html`, `maintenance.html`, and `customer.html` whenever making a functional change or fix. This helps verify that the latest update is live.

## Setup & Running
- **Local Development**: Open `index.html` or `maintenance.html` directly in any modern web browser. An internet connection is required for Firebase and YouTube APIs.
- **Updating Facts**:
  ```bash
  pip install anthropic
  export ANTHROPIC_API_KEY='your_key'
  python update_facts.py
  ```
- **Deployment**: Push changes to the `main` branch; GitHub Pages will deploy them automatically.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spataray)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/spataray)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
