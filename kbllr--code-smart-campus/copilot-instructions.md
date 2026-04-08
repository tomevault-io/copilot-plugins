## code-smart-campus

> This project is a real-time 3D visualization of a smart campus, integrating live data from Home Assistant. It provides an interactive interface to monitor and interact with various sensors and entities within a 3D representation of a building floorplan.

# Smart Campus Live Integration Project

This project is a real-time 3D visualization of a smart campus, integrating live data from Home Assistant. It provides an interactive interface to monitor and interact with various sensors and entities within a 3D representation of a building floorplan.

## Technologies Used

*   **Frontend:** Vanilla JavaScript
*   **Build Tool:** Vite
*   **3D Graphics:** Three.js
*   **Styling:** Tailwind CSS
*   **Home Automation Integration:** Home Assistant (via WebSockets)
*   **UI/Debugging:** Tweakpane, GSAP, Tween.js, Stats-gl
*   **SVG Processing:** Used for generating 3D room blocks from SVG floorplans.
*   **Language:** TypeScript (for type checking)

## Project Structure Highlights

*   `public/`: Static assets including 3D models, HDR environment maps, icons, and the main `index.html`.
*   `src/`: Contains the core application logic, organized into various modules:
    *   `src/main.js`: The main entry point of the application, handling initialization, Home Assistant connection, 3D scene setup, and UI interactions.
    *   `src/scene.js`: Manages the Three.js scene, including labels and Home Assistant WebSocket connection.
    *   `src/home_assistant/`: Modules for Home Assistant integration, state management, and WebSocket communication.
    *   `src/three/`: Three.js-specific utilities, such as `RoundedBlockGenerator.js`.
    *   `src/ui/components/`: Reusable UI components (atoms, molecules, organisms).
    *   `src/data/`: Data management, including `HistoryManager.js`.
    *   `src/utils/`: General utility functions, including `initCoordinator.js` for managing async dependencies.
    *   `src/debug/`: Debugging tools, including the `Debugger.js` which integrates with Tweakpane.
    *   `src/tools/`: Scripts for generating data like room and label registries.
*   `vite.config.js`: Vite configuration, including path aliases, proxy settings for API calls, and Tailwind CSS integration.
*   `package.json`: Defines project metadata, dependencies, and scripts.

## Building and Running

This project uses `npm` for package management and `Vite` for development and building.

### Prerequisites

*   Node.js (LTS version recommended)
*   npm (usually comes with Node.js)

### Installation

1.  **Install Dependencies:**
    ```bash
    npm install
    ```

### Development

1.  **Start Development Server:**
    This command will start the Vite development server, open the application in your browser, and automatically regenerate room and label registries.
    ```bash
    npm run dev
    ```
    The application will typically be available at `http://localhost:5173` (or another port if 5173 is in use).

### Production Build

1.  **Build for Production:**
    This command compiles the TypeScript code and bundles the application for production.
    ```bash
    npm run build
    ```
    The output will be in the `dist/` directory.

2.  **Preview Production Build:**
    To locally preview the optimized production build:
    ```bash
    npm run preview
    ```

### Linting

1.  **Run Linter:**
    This command checks the source code for style and quality issues using ESLint.
    ```bash
    npm run lint
    ```

### Utility Scripts

The project includes scripts to generate data used by the application:

*   **Generate Label Registry:**
    ```bash
    npm run generateLabelRegistry
    ```
*   **Generate Room Registry:**
    ```bash
    npm run generateRoomRegistry
    ```
    Note: `npm run dev` automatically runs `generateRoomRegistry` via `predev` hook.

## Development Conventions

*   **ES Modules:** The project uses JavaScript ES Modules (`"type": "module"`).
*   **TypeScript:** Type checking is performed using TypeScript.
*   **Path Aliases:** Vite is configured with various path aliases (e.g., `@`, `@components`, `@three`) to simplify imports.
*   **Asynchronous Initialization:** The `src/utils/initCoordinator.js` module is used to manage and synchronize asynchronous loading of different parts of the application (e.g., 3D models, Home Assistant data).
*   **Debugging:** An in-app debugger (`src/debug/Debugger.js`) is available, integrating with Tweakpane, and can be toggled with the 'D' key.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KBLLR)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KBLLR)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
