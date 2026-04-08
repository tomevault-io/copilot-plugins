## portfolio

> This is a modern, responsive personal portfolio website built for an Android Developer & Frontend Enthusiast (Biprangshu Das). It is a single-page application built with **Next.js (App Router)**, **React**, and **Tailwind CSS**. It showcases projects, experience, skills, and education, and includes a mock contact form that posts to a Next.js API route.

# Project Overview
This is a modern, responsive personal portfolio website built for an Android Developer & Frontend Enthusiast (Biprangshu Das). It is a single-page application built with **Next.js (App Router)**, **React**, and **Tailwind CSS**. It showcases projects, experience, skills, and education, and includes a mock contact form that posts to a Next.js API route.

## Main Technologies
*   **Framework:** Next.js (v15.x)
*   **Library:** React (v19.x)
*   **Styling:** Tailwind CSS (v3.x / v4.x via PostCSS)
*   **Icons:** Lucide React
*   **Deployment:** Designed for Vercel

## Architecture
*   **App Router:** Uses the `src/app` directory structure.
*   **Frontend:** The main layout and sections are housed in `src/app/page.js`, which acts as a large client component (`"use client"`) handling scroll spying, theme toggling (Dark/Light mode), and interactive state.
*   **Backend:** Contains a basic API route (`src/app/api/contact/route.js`) to handle POST requests from the contact form. Currently, it only logs the incoming data to the console and simulates a successful submission.

## Building and Running

You can use `npm`, `yarn`, or `pnpm` to manage the project.

*   **Install Dependencies:**
    ```bash
    npm install
    ```
*   **Run Development Server:**
    ```bash
    npm run dev
    ```
    *(Starts the server on `http://localhost:3000`)*
*   **Build for Production:**
    ```bash
    npm run build
    ```
*   **Start Production Server:**
    ```bash
    npm run start
    ```
*   **Run Linter:**
    ```bash
    npm run lint
    ```

## Development Conventions
*   **Styling:** Adhere to Tailwind CSS utility classes for styling. The project supports system-based and user-toggled Dark Mode, implemented via the `.dark` class on the root HTML element.
*   **State Management:** Standard React Hooks (`useState`, `useEffect`, `useRef`) are used for local component state.
*   **Component Structure:** Currently, most UI components (Hero, About, Experience, Projects, etc.) are co-located within `src/app/page.js`.
*   **Data:** The resume/portfolio data (skills, projects, education) is stored as a constant object (`resumeData`) at the top of `src/app/page.js`.
*   **Icons:** Import icons from `lucide-react`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jarvis1704)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jarvis1704)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
