## wallpaper-swipe

> A minimalistic React application inspired by Tinder's swipe mechanic, but for choosing wallpapers. Users can "pick" or "reject" wallpapers from a curated list.

# Wallpaper Swipe

A minimalistic React application inspired by Tinder's swipe mechanic, but for choosing wallpapers. Users can "pick" or "reject" wallpapers from a curated list.

## 🚀 Tech Stack

- **Frontend**: [React 19](https://react.dev/)
- **Build Tool**: [Vite 8](https://vite.dev/)
- **Styling**: [Tailwind CSS 4](https://tailwindcss.com/)
- **Animations**: [Framer Motion](https://www.framer.com/motion/)
- **Icons**: [Lucide React](https://lucide.dev/)

## 📂 Project Structure

- `src/components/`: UI components like `WallpaperCard`, `ActionButtons`, `Header`, `Footer`, `SessionSummary`.
- `src/hooks/`: Custom hooks for session management (`useWallpaperSession`) and keyboard navigation (`useKeyboardSwipe`).
- `src/App.jsx`: Main application entry point handling the flow between the swipe interface and the session summary.
- `public/`: Static assets (default wallpapers).

## ✨ Key Features

- **Tinder-like Swiping**: Smooth card animations for picking and rejecting wallpapers.
- **Keyboard Navigation**: Use arrow keys (Left/Right) to reject or pick wallpapers.
- **Session Summary**: View a grid of all picked wallpapers at the end of a session.
- **Liquid Glass Design**: Premium UI with soft gradients, glassmorphism, and smooth transitions.

## 🛠️ Development

### Commands

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Lint the project
npm run lint
```

## 📝 Configuration

- `vite.config.js`: Vite configuration with React and Tailwind plugins.
- `eslint.config.js`: ESLint rules for React.
- `package.json`: Project metadata and dependencies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshtyf)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joshtyf)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
