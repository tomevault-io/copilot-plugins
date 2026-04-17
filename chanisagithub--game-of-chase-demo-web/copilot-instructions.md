## game-of-chase-demo-web

> This project contains a Web Demo for "Game of Chase 101: Wild Cities", an interactive multiplayer game designed to simulate a Discord bot's mechanics. The entire application is self-contained within a single file (`game_of_chase_demo.html`).

# Game of Chase 101: Web Demo

## Project Overview

This project contains a Web Demo for "Game of Chase 101: Wild Cities", an interactive multiplayer game designed to simulate a Discord bot's mechanics. The entire application is self-contained within a single file (`game_of_chase_demo.html`). 

Key features demonstrated in this prototype include:
- A 6-dice rolling system with special outcomes (e.g., lottery, straight, craps).
- Faction-based turn logic (Bad Hustlas, Good Guys, Ugly Takers).
- Board path navigation with various space types (start, entry, jail, hospital, cemetery, etc.).
- Scalability to handle up to 14 simultaneous players, tracking individual positions, balances, and statuses.
- In-game events such as drawing jail cards and calculating multipliers.

## Building and Running

Since this is a standalone HTML file with embedded CSS and JavaScript, there are no build steps or dependencies required.

**To run the demo:**
Simply open `game_of_chase_demo.html` in any modern web browser.

## Development Conventions

- **Architecture:** Single-file application. All HTML, CSS, and JavaScript are bundled together in `game_of_chase_demo.html`.
- **Technologies:** Vanilla HTML, CSS, and JavaScript. No external libraries or frameworks (like React, Vue, or TailwindCSS) are used.
- **Styling:** Custom vanilla CSS utilizing CSS variables (custom properties) for theming and consistent spacing/sizing.
- **State Management:** Game state (players, board layout, current turn, game log) is managed via global JavaScript variables and updated through direct DOM manipulation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chanisagithub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
