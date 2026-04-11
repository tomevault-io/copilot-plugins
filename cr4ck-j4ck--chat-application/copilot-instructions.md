## chat-application

> This is a full-stack chat application built with the MERN stack (MongoDB, Express.js, React, Node.js) and TypeScript. It uses Socket.IO for real-time communication between the client and the server.

# Project Overview

This is a full-stack chat application built with the MERN stack (MongoDB, Express.js, React, Node.js) and TypeScript. It uses Socket.IO for real-time communication between the client and the server.

## Backend

The backend is a Node.js application built with Express.js and TypeScript. It uses MongoDB as the database and Mongoose as the ODM. It provides a RESTful API for user authentication and a WebSocket API for real-time messaging.

**Key Technologies:**

*   **Framework:** Express.js
*   **Language:** TypeScript
*   **Database:** MongoDB
*   **ODM:** Mongoose
*   **Real-time Communication:** Socket.IO
*   **Authentication:** JWT (jsonwebtoken) and bcrypt

## Frontend

The frontend is a React application built with Vite and TypeScript. It uses Zustand for state management, React Router for routing, and Tailwind CSS for styling. It communicates with the backend via a RESTful API and a WebSocket connection.

**Key Technologies:**

*   **Framework:** React
*   **Language:** TypeScript
*   **Build Tool:** Vite
*   **Styling:** Tailwind CSS
*   **Routing:** React Router
*   **State Management:** Zustand
*   **Form Handling:** React Hook Form with Zod for validation
*   **Real-time Communication:** Socket.IO Client

# Building and Running

## Backend

To build and run the backend, you need to have Node.js and npm installed.

1.  **Install dependencies:**
    ```bash
    cd Backend
    npm install
    ```

2.  **Build the application:**
    ```bash
    npm run build
    ```

3.  **Run the application in development mode:**
    ```bash
    npm run dev
    ```

4.  **Run the application in production mode:**
    ```bash
    npm start
    ```

## Frontend

To build and run the frontend, you need to have Node.js and npm installed.

1.  **Install dependencies:**
    ```bash
    cd Frontend
    npm install
    ```

2.  **Run the application in development mode:**
    ```bash
    npm run dev
    ```

3.  **Build the application:**
    ```bash
    npm run build
    ```

4.  **Preview the production build:**
    ```bash
    npm run preview
    ```

# Development Conventions

*   **Code Style:** The project uses ESLint to enforce a consistent code style. You can run the linter with the `npm run lint` command in the `Frontend` directory.
*   **Testing:** There are no tests in the project yet. This is something that could be added in the future.
*   **Commits:** There is no specific commit message format enforced in the project. However, it is recommended to write clear and concise commit messages that explain the changes you have made.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cr4ck-j4ck)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cr4ck-j4ck)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
