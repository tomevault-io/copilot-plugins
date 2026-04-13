## sg-bank-interest-cal

> The **SG Fund Allocation Optimizer** is a client-side web application designed to help users in Singapore maximize their monthly interest earnings by calculating the optimal allocation of funds across various bank accounts (UOB, SC, DBS, CIMB). It runs entirely in the browser using modern web technologies.

# SG Fund Allocation Optimizer - Project Context

## Project Overview

The **SG Fund Allocation Optimizer** is a client-side web application designed to help users in Singapore maximize their monthly interest earnings by calculating the optimal allocation of funds across various bank accounts (UOB, SC, DBS, CIMB). It runs entirely in the browser using modern web technologies.

## Tech Stack & Architecture

*   **Frontend:** HTML5, JavaScript (ES6+ Modules), Tailwind CSS.
*   **Tooling:** Node.js, PostCSS, Autoprefixer.
*   **Testing:** Jest, Babel.
*   **Architecture:** Client-side SPA (Single Page Application). Logic is separated into calculation engines and UI handling.
*   **Documentation:** Maintains a `memory-bank/` directory for active context, product details, and technical patterns.

## Key Files & Directories

*   `src/logic/`: Core calculation logic (e.g., `allocation-engine.js`, `calculator.js`).
*   `src/config/`: Configuration files (e.g., `bank-rates.js`) separating data from logic.
*   `src/ui/`: UI interaction logic (`script.js`).
*   `__tests__/`: Unit tests for the logic engines.
*   `index.html`: The main entry point for the application.
*   `style.css`: Compiled Tailwind CSS output.
*   `src/input.css`: Source CSS file for Tailwind.
*   `memory-bank/`: Comprehensive documentation of the project's state and context.

## Development Workflow

### Building and Running

1.  **Install Dependencies:**
    ```bash
    npm install
    ```

2.  **Start Development Server:**
    ```bash
    npm start
    ```
    This serves the application locally using `http-server` on port 8080.

3.  **Build CSS:**
    ```bash
    npm run build
    ```
    Compiles `src/input.css` to `src/output.css` using Tailwind CSS.

4.  **Full Deployment for Inspection:**
    To build and run the server in the background for inspection (e.g., on port 8080):
    ```bash
    npm run build; npm start
    ```

5.  **Run Tests:**
    ```bash
    npm test
    ```
    Executes the Jest test suite located in `__tests__/`.

### Coding Conventions

*   **Modular JavaScript:** Use ES Modules (`import`/`export`) to organize code.
*   **Separation of Concerns:** Keep calculation logic, configuration data, and UI manipulation separate.
*   **Utility-First CSS:** Use Tailwind CSS for styling. Avoid writing custom CSS unless necessary.
*   **Testing:** Write unit tests for all new calculation logic.
*   **Documentation:** Update the `memory-bank/` files when making significant architectural or product changes.

## Memory Bank

This project uses a "Memory Bank" pattern to maintain context. Refer to `memory-bank/activeContext.md` for the current status and `memory-bank/techContext.md` for deeper technical details.

## Deployment

The project is deployed as a static site (e.g., GitHub Pages). The `index.html` file works directly when served, provided the CSS is built.

## Current Default Settings (as of Jan 20, 2026)

The following defaults are set in `index.html`:
1.  **UOB One**: Included, with "Min. S$500 card spend" and "Salary Credit" enabled.
2.  **UOB Stash**: NOT included.
3.  **SC Bonus$aver**: Included, with "Salary Credit" enabled (Card Spend disabled).
4.  **OCBC 360**: NOT included.
5.  **DBS Multiplier**: Included, set to "1 Category" and volume "≥ S$500 to < S$15k".
6.  **CIMB FastSaver**: Included.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alfred1137)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alfred1137)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
