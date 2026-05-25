## snake-game-html-with-claude

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an advanced Snake Game implemented with modular JavaScript architecture. The game features modern web technologies including HTML5 Canvas for rendering, comprehensive game features, and a glassmorphism UI design with Thai language interface elements. The project is based on and extends the [Devahoy Snake Game tutorial](https://www.devahoy.com/blog/2025/create-snake-game-html).

## Development Commands

### Running the Game
```bash
# Start development server with hot reload
bun run dev

# Build for production
bun run build

# Preview built application
bun run preview
```

### Package Management
- Uses **Bun** as package manager (bun.lock present)
- Only dependency is Vite for development tooling

## Architecture

### Modular Structure
- **Modular JavaScript** - Game logic split into separate modules (game.js, ui.js, ai.js, powerups.js)
- **Canvas-based rendering** - Flexible game area with adjustable sizes (300x300, 400x400, 500x500)
- **ES6 Modules** - Modern JavaScript module system
- **Separation of concerns** - Clear separation between game logic, UI, and features

### Game Implementation Details
- **Variable frame rate** - Adjustable based on difficulty settings
- **Controls**: Arrow keys for movement
- **Game loop**: Uses `requestAnimationFrame` for smooth animation
- **Advanced collision detection**: Grid-based boundary, self-collision, and AI collision
- **Comprehensive scoring**: Points, high scores, and leaderboard system
- **Multiple game modes**: Single player, AI mode, various difficulty levels

### Advanced Features
- **AI System**: Computer opponent with adjustable difficulty
- **Power-ups**: Special items that enhance gameplay
- **Theme System**: Multiple color themes (default, dark, ocean, sunset)
- **Settings System**: Comprehensive game customization
- **Leaderboard**: Local storage-based score tracking
- **Sound System**: Toggle-able game audio

### UI/UX Features
- **Full Thai interface**: Complete Thai language support
- **Glassmorphism design**: Modern backdrop blur effects and gradient backgrounds
- **Google Fonts**: Uses "Itim" font family for Thai text
- **Responsive layout**: Flexbox centering with viewport meta tag
- **Modal system**: Settings, leaderboard, and score input modals

## File Structure

- `index.html` - Main HTML entry point
- `src/main.js` - Application initialization
- `src/game.js` - Core game logic and mechanics
- `src/ui.js` - User interface management
- `src/ai.js` - AI opponent system
- `src/powerups.js` - Power-up system implementation
- `src/styles.css` - Game styling and themes
- `package.json` - Project config and scripts
- `public/vite.svg` - Favicon
- `README.md` - Project documentation in Thai

## Development Notes

- **Modular architecture**: Game logic is split into focused modules for maintainability
- **Modern JavaScript**: Uses ES6+ features and module system
- **Vite integration**: Fast development server with hot reload
- **Development workflow**: Edit source files, Vite handles bundling and serving
- **Build process**: `bun run build` creates optimized production bundle

## Documentation

- **README.md**: Comprehensive Thai documentation covering all features
- **CLAUDE.md**: Development guidance for Claude Code
- **Reference**: Based on [Devahoy Snake Game tutorial](https://www.devahoy.com/blog/2025/create-snake-game-html)

---
> Source: [Phonbopit/snake-game-html-with-claude](https://github.com/Phonbopit/snake-game-html-with-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
