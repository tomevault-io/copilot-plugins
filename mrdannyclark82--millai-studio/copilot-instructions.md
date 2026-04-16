## millai-studio

> This project is "Milla Rayne," a "Devoted Companion" AI Studio application. It's built as a modern web application using a stack that includes:

# GEMINI.md - Millai Studio

## Project Overview

This project is "Milla Rayne," a "Devoted Companion" AI Studio application. It's built as a modern web application using a stack that includes:

*   **Framework:** React with TypeScript
*   **Build Tool:** Vite
*   **Styling:** Tailwind CSS
*   **Core Logic:** The application integrates with the Gemini API for its AI capabilities. It also uses TensorFlow.js for on-device model execution.

The application is structured as a single-page application with a rich user interface that includes a variety of features, such as:

*   **Core:** A chat interface, a code sandbox, and voice/vision modes.
*   **Creative Suite:** A creative studio, podcast player, and video generator.
*   **Productivity:** A morning sync feature, task manager, and a "Memory Galaxy" for visualizing past interactions.
*   **Intelligence:** The ability to enable "Thinking Mode," "Deep Search," and "Maps Grounding."
*   **System & Data:** Features for managing external databases, backups, and offline models.

## Building and Running

To get the project running locally, follow these steps:

1.  **Install Dependencies:**
    ```bash
    npm install
    ```

2.  **Set Environment Variables:**
    Create a `.env.local` file in the root of the project and add the following, replacing the placeholder with your Gemini API key:
    ```
    GEMINI_API_KEY=your_api_key_here
    ```

3.  **Run the Development Server:**
    ```bash
    npm run dev
    ```

4.  **Build for Production:**
    ```bash
    npm run build
    ```

5.  **Preview the Production Build:**
    ```bash
    npm run preview
    ```

## Development Conventions

*   **Component-Based Architecture:** The application is organized into a `components` directory, with each component responsible for a specific piece of the UI.
*   **Service Layer:** Business logic and external API interactions are handled in a `services` directory, promoting separation of concerns.
*   **TypeScript:** The entire codebase is written in TypeScript, ensuring type safety and improved developer experience.
*   **Styling:** Tailwind CSS is used for styling, with theme-related utilities in `utils/theme.ts`.
*   **State Management:** The main `App.tsx` component manages the majority of the application's state.
*   **Modularity:** The application is highly modular, with features like `CreativeStudio`, `Sandbox`, and `LiveSession` encapsulated in their own components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrdannyclark82) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
