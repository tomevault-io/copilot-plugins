## note-tui

> `note-tui` is a Terminal User Interface (TUI) application designed for efficient note management directly from the command line. It provides a clean interface with a split-pane view, allowing users to browse a list of notes and preview their content with full Markdown syntax highlighting.

# Project Overview: note-tui

`note-tui` is a Terminal User Interface (TUI) application designed for efficient note management directly from the command line. It provides a clean interface with a split-pane view, allowing users to browse a list of notes and preview their content with full Markdown syntax highlighting.

## Technologies Used

- **Runtime:** [Bun](https://bun.sh/) (used for development and building)
- **Frontend:** React (rendered in the terminal using [Ink 6](https://www.npmjs.com/package/ink))
- **Language:** TypeScript
- **State Management:** [Zustand](https://zustand-bear.github.io/blog/) (with `immer` middleware)
- **Markdown Rendering:** `marked` and `marked-terminal`
- **TUI Components:** `ink-scroll-list`, `ink-scroll-view`, `ink-text-input`
- **Utilities:** `dayjs` (date handling), `fuse.js` (fuzzy searching), `clsx` (class name merging), `nanoid`
- **Linting/Formatting:** `xo`, `prettier`
- **Testing:** `ava`

## Architecture

The application follows a component-based architecture:

- **`source/cli.tsx`**: The entry point, which renders the main `App` component using Ink and handles terminal cleanup on exit.
- **`source/app.tsx`**: The main application component, responsible for the overall layout (list pane, preview pane, instruction area) and managing terminal-specific escape sequences (alternate screen, mouse tracking).
- **`source/components/`**:
  - `ListPane`: Renders the list of notes with scroll support.
  - `PreviewPane`: Renders the Markdown content of the selected note.
  - `Instruction`: Displays available keybindings based on the current mode.
- **`source/hooks/`**:
  - `useNavigation`: Handles global keybindings and application mode transitions.
  - `usePreviewNavigation`: Manages scrolling and navigation within the preview pane.
  - `useDimension`: Tracks terminal window dimensions.
  - `useDebounce`: Utility for debouncing values (e.g., search input).
- **`source/store/`**: Global state management using Zustand, partitioned into:
  - `index.ts`: Main store for notes, selection, and application mode.
  - `type.ts`: TypeScript definitions for store state and actions.
- **`source/helper/`**:
  - `file.ts`: Core logic for file system operations (CRUD for notes and metadata in `~/.notes`).
  - `editor.ts`: Logic for spawning external editors (nvim, vim, vi) to edit notes.
  - `search.ts`: Fuzzy search implementation using `fuse.js`.
  - `theme.ts`: Persistent theme storage and retrieval.
- **`source/theme/`**: Manages the application's theming system, defining multiple color schemes (e.g., Catppuccin, Nord, Tokyo Night).

## Building and Running

The project primarily uses **Bun** for its workflow:

1. **Installation:**
   ```bash
   bun install
   ```

2. **Development:** Runs the application in watch mode.
   ```bash
   bun dev
   ```

3. **Running from Source:**
   ```bash
   bun start
   ```

4. **Building Binaries:** Compiles the application into standalone executables for multiple platforms (Linux, macOS).
   ```bash
   bun run build:binary
   ```

## Testing and Quality

- **Linting:** `npx xo`
- **Formatting:** `npx prettier --check .`
- **Tests:** `npx ava` (Note: Run `bun test` if the project is configured for it, but currently uses `ava`).

## Keybindings

`note-tui` features Vim-like navigation:
- `j`/`k` or arrows for movement.
- `h`/`l` or `Tab` to switch focus between List and Preview panes.
- `e` or `Enter` to edit a note in your default editor.
- `n` to create a new note.
- `/` or `s` to enter search mode.
- `t` to view trash; `r` to restore; `d` to archive or delete.
- `<`/`>` to cycle through themes.

---
> Source: [Pansther/note-tui](https://github.com/Pansther/note-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
