## nannipy-com

> This project is a modern and interactive personal portfolio website for Giovanni Battista Pernazza, a software engineer. It's built with Next.js and TypeScript, and it showcases his projects and work experience in a clean, terminal-inspired interface. The portfolio features a dark mode, responsive design, and smooth animations.

# GEMINI.md

## Project Overview

This project is a modern and interactive personal portfolio website for Giovanni Battista Pernazza, a software engineer. It's built with Next.js and TypeScript, and it showcases his projects and work experience in a clean, terminal-inspired interface. The portfolio features a dark mode, responsive design, and smooth animations.

**Key Technologies:**

*   **Framework:** Next.js
*   **Language:** TypeScript
*   **Styling:** Tailwind CSS
*   **Deployment:** Vercel

**Architecture:**

The project follows a standard Next.js project structure. The `src` directory contains the core application logic, including:

*   `src/app`: The main application router, with pages and layouts.
*   `src/components`: Reusable React components.
*   `src/lib`: Data for projects and work experience.
*   `src/styles`: Global CSS styles.
*   `src/types`: TypeScript type definitions.

## Building and Running

### Prerequisites

*   Node.js >= 14.0.0

### Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/nannipy/nannipy.git
    ```
2.  **Navigate to the project directory:**
    ```bash
    cd nannipy
    ```
3.  **Install dependencies:**
    ```bash
    npm install
    ```

### Running the Project

*   **Development:**
    ```bash
    npm run dev
    ```
    This will start the development server at `http://localhost:3000`.

*   **Production:**
    ```bash
    npm run build
    npm run start
    ```
    This will build the application for production and start the production server.

### Other Commands

*   **Linting:**
    ```bash
    npm run lint
    ```
    This will check the code for any linting errors.

## Development Conventions

*   **Styling:** The project uses Tailwind CSS for styling. Utility classes are preferred over custom CSS.
*   **Components:** Reusable components are located in the `src/components` directory.
*   **Data:** Project and work experience data is stored in `src/lib/projects.ts` and `src/lib/work.ts` respectively.
*   **Animations:** The portfolio uses Framer Motion for animations.
*   **Easter Eggs:** The footer contains a hidden animation that is triggered by clicking the logo twice.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nannipy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nannipy)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
