## web-desktop

> This project is a web-based desktop environment with a client-server architecture. It provides a graphical user interface in the browser for managing applications, files, and system resources.

# GEMINI.md

## Project Overview

This project is a web-based desktop environment with a client-server architecture. It provides a graphical user interface in the browser for managing applications, files, and system resources.

The architecture consists of:

*   **Backend**: A Node.js server built with Express and TypeScript. It uses WebSockets for real-time communication and provides RESTful APIs for interacting with the file system, Docker containers, and system information.

*   **Frontend (Modern)**: A React-based single-page application built with Vite and TypeScript. It uses Tailwind CSS for styling and features a window manager with both tiling and floating modes. This is the primary, actively developed frontend.

*   **Frontend (Legacy)**: A simple, static frontend built with vanilla JavaScript, HTML, and CSS. It is served by the backend and appears to be an older version of the application.

## Building and Running

### Backend

To build and run the backend server:

```bash
cd backend
npm install
npm run dev
```

The server will start on `http://localhost:3001`.

### Frontend (Modern)

To build and run the modern React-based frontend:

```bash
cd frontend
npm install
npm run dev
```

The frontend development server will start on a different port (usually `http://localhost:5173`) and will proxy API requests to the backend.

### Frontend (Legacy)

To access the legacy frontend, run the backend server and navigate to `http://localhost:3001` in your browser.

## Development Conventions

*   **Code Style**: The project uses ESLint for code linting in the frontend. The configuration can be found in `frontend/eslint.config.js`.

*   **Type Safety**: Both the frontend and backend use TypeScript for static typing.

*   **API**: The backend provides a RESTful API for various system-level operations. The API routes are defined in `backend/src/routes/`.

*   **State Management**: The frontend uses React Context for managing the state of the windowing system.

*   **Component-Based Architecture**: The React frontend is built using a component-based architecture. The main components are located in `frontend/src/components/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ghively) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
