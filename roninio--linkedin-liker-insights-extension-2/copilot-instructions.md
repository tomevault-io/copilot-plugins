## linkedin-liker-insights-extension-2

> **1. Core Functionality:**

### Project: LinkedIn Liker Insights Extension

**1. Core Functionality:**
This is a Chrome extension designed to extract and analyze data from LinkedIn. Its primary purpose is to provide insights into users who have "liked" a specific LinkedIn post.

**2. Key Features:**
- **Liker Extraction:** Scans a LinkedIn post and extracts a list of users who have liked it, capturing their name, title, and profile URL.
- **Profile Analysis:** Can perform a deeper analysis of individual profiles to extract more detailed information like location, the "About" section, and professional experience.
- **Data Export:** Allows the user to export the collected data into CSV or JSON format.
- **UI:** Features a modern user interface with a "glassmorphism" design, and includes both a popup and a full-tab view.

**3. Technical Stack:**
- **Frontend:** Built with React and TypeScript.
- **Styling:** Uses Tailwind CSS.
- **Build Tool:** Vite is used for the development server and building the extension.
- **Platform:** It's a Chrome Extension using Manifest V3.

**4. How it Works:**
- A `content_script.ts` is injected into LinkedIn pages. This script is responsible for finding the "likes" button on a post, clicking it to open the modal, and then scrolling through the list of likers to extract their information.
- The main application logic resides in `App.tsx` and various React components.
- It uses Chrome's local storage to persist the data it collects.
- The `geminiService.ts` suggests an integration with the Google Gemini API to perform analysis on the collected profile data, though it notes that an API key is required.

**5. Project Structure:**
- `components/`: Contains the React components for the UI.
- `services/`: Includes the `geminiService.ts` for AI-powered analysis.
- `content_script.ts`: The script that interacts with the LinkedIn website.
- `background.ts`: The service worker for the extension.
- `manifest.json`: The configuration file for the Chrome extension.
- `App.tsx`, `index.tsx`: The main entry points for the React application.

In essence, this project is a data extraction and analysis tool for LinkedIn, packaged as a user-friendly Chrome extension. It leverages modern web technologies and has the capability to integrate with AI for deeper insights.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/roninio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
