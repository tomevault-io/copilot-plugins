## gemini-time-photo

> This is a React and TypeScript application named "ChronoSnap," designed to leverage Google Gemini AI models for image manipulation and analysis. The application allows users to take a selfie and either "time travel" by editing the image into various historical or future contexts using `gemini-2.5-flash-image`, or analyze an image in detail using `gemini-3-pro-preview`. The project is built with Vite.

# Project Overview

This is a React and TypeScript application named "ChronoSnap," designed to leverage Google Gemini AI models for image manipulation and analysis. The application allows users to take a selfie and either "time travel" by editing the image into various historical or future contexts using `gemini-2.5-flash-image`, or analyze an image in detail using `gemini-3-pro-preview`. The project is built with Vite.

Key technologies:
*   **Frontend:** React, TypeScript
*   **Build Tool:** Vite
*   **AI Integration:** Google Gemini API (`@google/genai`)
*   **UI Components:** `lucide-react` (icons)
*   **Styling:** Tailwind CSS (inferred)

# Building and Running

To get the ChronoSnap application up and running locally, follow these steps:

1.  **Install Dependencies:**
    ```bash
    npm install
    ```
2.  **Set up Gemini API Key:**
    *   Create a `.env.local` file in the project root.
    *   Add your Gemini API key to this file:
        ```
        GEMINI_API_KEY=YOUR_GEMINI_API_KEY
        ```
3.  **Run the Application (Development Mode):**
    ```bash
    npm run dev
    ```
    This will start the development server, typically accessible at `http://localhost:3000`.

4.  **Build for Production:**
    ```bash
    npm run build
    ```
    This command compiles the application for production, outputting static files to the `dist` directory.

5.  **Preview Production Build:**
    ```bash
    npm run preview
    ```
    This command serves the production build locally for testing.

# Development Conventions

*   **Language:** TypeScript is used for all application logic, ensuring type safety and improved code quality.
*   **Framework:** React is utilized for building the user interface, with a component-based architecture found in the `components` directory.
*   **Build System:** Vite is configured for fast development and efficient bundling.
*   **API Integration:** The `@google/genai` library is used to interact with Google Gemini AI models.
*   **Environment Variables:** Sensitive information like the Gemini API key is managed through environment variables, loaded by Vite and accessed via `process.env.GEMINI_API_KEY`.
*   **Image Handling:** Images are primarily handled as base64 encoded strings for transmission to and from the Gemini API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LittleBreak)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LittleBreak)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
