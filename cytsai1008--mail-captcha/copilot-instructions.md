## mail-captcha

> This project is a full-stack web application that demonstrates how to protect `mailto:` links from bots using Google reCAPTCHA v3. It consists of a separate backend and frontend.

# Project Overview

This project is a full-stack web application that demonstrates how to protect `mailto:` links from bots using Google reCAPTCHA v3. It consists of a separate backend and frontend.

*   **Backend:** Developed with Node.js, Express, and Axios. Its primary role is to handle reCAPTCHA v3 token verification by communicating with the Google reCAPTCHA API. It uses `dotenv` for managing environment variables, including the reCAPTCHA secret key and the target email address for `mailto:` links.
*   **Frontend:** A client-side application built with HTML, CSS, and JavaScript. It integrates with reCAPTCHA v3 to generate tokens, sends these tokens to the backend for verification, and upon successful verification, redirects the user to a dynamically generated `mailto:` link.

## Building and Running

### Prerequisites

*   Node.js and npm installed.
*   A Google reCAPTCHA v3 site key and secret key.

### Backend (Node.js)

1. **Navigate to the backend directory:**
    ```bash
    npm install --prefix backend
    ```
2. **Configure environment variables:**
    Create a `.env` file in the `backend` directory with the following content, replacing the placeholder values:
    ```
    RECAPTCHA_SECRET_KEY=YOUR_RECAPTCHA_SECRET_KEY
    MAIL_TO_EMAIL=your-email@example.com
    ```
3. **Start the backend server:**
    ```bash
    cd backend
    npm start
    # or
    node server.js
    ```
    The server will typically run on `http://localhost:3000`.

### Frontend (HTML/CSS/JavaScript)

1. **Navigate to the frontend directory:**
    ```bash
    npm install --prefix frontend
    ```
2. **Configure reCAPTCHA site key:**
    Open `index.html` and `script.js` in the `frontend` directory and replace `YOUR_SITE_KEY` with your actual reCAPTCHA v3 site key.
3. **Serve the frontend:**
    ```bash
   cd frontend
    npm start
    # or, if http-server is globally installed or run directly:
    npx http-server -c-1
    ```
    Open `http://localhost:8080` (or the port indicated by `http-server`) in your web browser.

## Development Conventions

The project enforces code quality and consistency using various linting tools:

*   **JavaScript (Backend & Frontend):** ESLint is used for linting JavaScript code.
    *   Backend configuration: `backend/eslint.config.js`
    *   Frontend configuration: `frontend/eslint.config.mjs`
    *   Run linting: `npm run lint` (in respective directories)
*   **CSS (Frontend):** Stylelint is used for linting CSS files.
    *   Configuration: `frontend/.stylelintrc.json`
    *   Run linting: `npm run lint:css` (in `frontend` directory)
*   **HTML (Frontend):** HTMLHint is used for linting HTML files.
    *   Configuration: `frontend/.htmlhintrc`
    *   Run linting: `npm run lint:html` (in `frontend` directory)

The `frontend` directory also has a combined linting script: `npm run lint` which runs `lint:js`, `lint:css`, and `lint:html`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cytsai1008)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cytsai1008)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
