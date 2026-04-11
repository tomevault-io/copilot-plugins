## calchat

> CalChat is a modern, monochromatic web application designed for frictionless calorie and macronutrient tracking using voice commands. It now uses the browser's built-in Web Speech API for completely free voice recognition.

# CalChat: Minimalist Voice-Activated Macro Tracking

CalChat is a modern, monochromatic web application designed for frictionless calorie and macronutrient tracking using voice commands. It now uses the browser's built-in Web Speech API for completely free voice recognition.

## 🚀 Key Technologies

-   **Frontend**: React 19 (Vite)
-   **Language**: TypeScript
-   **Speech Recognition**: Browser Web Speech API (100% free, no API keys)
-   **Icons**: Lucide React
-   **Storage**: Browser LocalStorage (Persistence layer)
-   **Styling**: Modern Monochromatic CSS (Tailwind-like utility approach in standard CSS)

## ✨ Features

-   **Voice-First Interface**: Tap the microphone to record what you ate.
-   **Free Speech Recognition**: Uses the browser's built-in Web Speech API - no API keys required!
-   **Automatic Macro Parsing**: Automatically identifies known foods and calculates Calories, Protein, Carbs, and Fat.
-   **Minimalist Dashboard**: A clean, black-and-white UI focusing on the data that matters.
-   **Persistent Tracking**: All entries are stored locally in the browser, ensuring data stays private and accessible.
-   **Daily Reset**: Easily clear the current day's progress for a fresh start.
-   **Privacy-Friendly**: All speech processing happens in the browser - no data sent to external servers.

## 🛠️ Project Structure

-   `App.tsx`: Main application container and logic.
-   `components/`: UI components (Header, MacroDisplay, VoiceInput, etc.).
-   `hooks/`: Custom React hooks, including `useSpeechRecognition.ts` which handles browser speech recognition.
-   `services/`: Core logic for data persistence (`dbService.ts`).
-   `utils/`: Utility functions like the food parser.
-   `constants.tsx`: Static data including the hardcoded food database and default values.
-   `types.ts`: TypeScript interfaces and types.

## 🏁 Getting Started

1.  **Clone & Install**:
    ```bash
    npm install
    ```
2.  **Run Development Server**:
    ```bash
    npm run dev
    ```

**No configuration needed!** The app uses the browser's built-in Web Speech API, so it works immediately without any API keys.

## 🧠 How It Works

The app uses the **Web Speech API** built into modern browsers (Chrome, Edge, Safari, Firefox). When you tap the microphone:

1. Your browser's speech recognition engine activates
2. Audio is processed locally or by your browser vendor (depending on browser)
3. Transcribed text is matched against our food database
4. Macros are calculated and stored locally

This approach is completely free, privacy-friendly, and works offline in some browsers!

---

*Built with ❤️ for rapid health tracking.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jarmen423)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jarmen423)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
