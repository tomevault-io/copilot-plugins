## mscv-js

> This document provides a comprehensive, in-depth overview of the MSCV portfolio project. It is designed to be understandable by both human developers and AI assistants, detailing the architecture, components, data flow, and operational procedures.

# Gemini Project Documentation: MSCV Portfolio

This document provides a comprehensive, in-depth overview of the MSCV portfolio project. It is designed to be understandable by both human developers and AI assistants, detailing the architecture, components, data flow, and operational procedures.

---

## 1. Project Overview

MSCV is a modern single-page application (SPA) portfolio website. It showcases projects and skills, and includes a contact form. The frontend is built with **React** and **Vite**, and the backend is a **Node.js/Express** serverless function. The entire application is designed for and deployed on **Vercel**, utilizing **Vercel Postgres (Neon)** for the database.

### Key Technologies
- **Frontend Framework:** React
- **Build Tool:** Vite
- **Styling:** Bootstrap, Custom CSS (`style.css`, `responsive.css`)
- **Routing:** React Router DOM
- **Backend:** Node.js with Express.js (as a Vercel Serverless Function)
- **Database:** Vercel Postgres (Neon) via the `pg` driver
- **Deployment:** Vercel
- **Environment Variables:** `dotenv` for local development

---

## 2. System Architecture

The project employs a decoupled architecture, consisting of two main parts that work together:

1.  **Frontend (Static SPA):** A React application built by Vite. The user's browser downloads this application, which then handles UI rendering and routing. It makes API calls to the backend to fetch dynamic data.
2.  **Backend (Serverless API):** An Express.js application that runs as a Vercel Serverless Function. It exposes several API endpoints to interact with the database and handle tasks like sending emails.

### How It Works on Vercel
- When a user visits the website, Vercel serves the static, pre-built React application from its `dist` directory.
- When the React app makes an API request (e.g., to `/api/portfolio`), Vercel's routing rules (`vercel.json`) direct that request to the backend serverless function.
- The serverless function processes the request, queries the Vercel Postgres database, and returns a JSON response to the frontend.

---

## 3. Directory Structure

Here is a breakdown of the key files and directories:

```
/
├── .env                # Local environment variables (pulled from Vercel). Not committed.
├── db.js               # Configures and exports the PostgreSQL database connection pool.
├── package.json        # Project dependencies and scripts.
├── vercel.json         # Vercel build and routing configuration.
├── vite.config.js      # Vite configuration, including the local dev proxy.
│
├── public/             # Static assets directly copied to the build output.
│
├── server/
│   └── server.js       # The backend Express.js application (API endpoints).
│
└── src/                # The React application source code.
    ├── App.jsx         # Main component, contains the core routing structure.
    ├── main.jsx        # The entry point for the React application.
    │
    ├── components/     # Reusable React components (e.g., Layout, Hero, PortfolioShowcase).
    ├── pages/          # Page components rendered by the router (e.g., HomePage).
    └── styles/         # CSS stylesheets.
```

---

## 4. Frontend (React)

The frontend is a standard Vite-powered React application.

### Routing
- **`react-router-dom`** is used for all client-side routing.
- **`src/App.jsx`** defines the routes.
- A `<Layout />` component wraps all pages, providing a consistent structure (e.g., navigation, footer).
- The primary route is the index (`/`), which renders the `<HomePage />` component. Other pages like `ServicePage` exist but are not yet integrated into the router.

### Key Components
- **`pages/HomePage.jsx`**: The main landing page, composed of several smaller components.
- **`components/Layout.jsx`**: Provides the overall page structure.
- **`components/PortfolioShowcase.jsx`**: A key component responsible for fetching and displaying portfolio projects from the backend.
- **`components/contact.jsx`**: A form that allows users to send messages via the `/api/send-email` endpoint.
- **`components/hero.jsx`**: The main "hero" section of the home page.

### Styling
The project uses a combination of styling methods:
- **`Bootstrap`**: For base components and grid system.
- **`src/styles/style.css`**: The main stylesheet for custom application styles.
- **`src/styles/responsive.css`**: Contains media queries to handle different screen sizes.
- **`src/styles/override.css`**: Used for overriding default styles from Bootstrap or other libraries.

---

## 5. Backend (Node.js/Express API)

The backend is a single Express.js file (`server/server.js`) that runs as a serverless function on Vercel.

### API Endpoints
- **`GET /api`**: A test endpoint to confirm the backend is running.
- **`GET /api/test-db`**: A utility endpoint to test the database connection.
- **`GET /api/categories`**: Fetches and returns all project categories from the `web_categories` table in the database.
- **`GET /api/portfolio`**: Fetches and returns all projects from the `portfolio` table in the database.
- **`POST /api/send-email`**: Handles the contact form submission. It uses `nodemailer` to send an email with the user's message. Requires `EMAIL_USER` and `EMAIL_PASS` environment variables.

---

## 6. Database

- The application uses a **Vercel Postgres (Neon)** database.
- The `pg` library is used to connect to the database.
- **`db.js`** creates and configures a connection pool. It is configured to use the `POSTGRES_URL` environment variable for the connection string and enables SSL, which is required by Neon.

### Inferred Schema
Based on the queries in `server.js`, the database contains at least the following tables:
- **`web_categories`**: Stores information about project categories.
- **`portfolio`**: Stores information about individual portfolio projects.

---

## 7. Configuration and Deployment

### Environment Variables
The following environment variables are required for the application to function correctly:
- **`POSTGRES_URL`**: The full connection string for the Vercel Postgres database.
- **`EMAIL_USER`**: The username for the email account used by `nodemailer` to send emails.
- **`EMAIL_PASS`**: The password for the email account.
- **`VITE_UNDER_CONSTRUCTION`**: (Optional) Set to `'true'` to display a loader for the entire site.

### Local Development
- The `vite.config.js` file configures a proxy. Any request from the frontend to `/api` is automatically forwarded to the backend server running on `http://localhost:3001`. This mimics the production routing behavior.

### Vercel Deployment
- The `vercel.json` file orchestrates the deployment.
- It defines two build processes: one for the Node.js backend (`@vercel/node`) and one for the React frontend (`@vercel/static-build`).
- It sets up rewrite rules to direct traffic appropriately: `/api/*` requests go to the serverless function, and all other requests are served the static frontend.

---

## 8. Local Development Setup

To run the project locally, you must have the [Vercel CLI](https://vercel.com/docs/cli) installed.

1.  **Install Dependencies:**
    ```bash
    npm install
    ```

2.  **Link to Vercel Project:** (One-time setup)
    ```bash
    vercel link
    ```

3.  **Pull Environment Variables:** This creates the `.env` file with the correct database credentials for your current git branch.
    ```bash
    vercel env pull .env
    ```

4.  **Run the Development Servers:**
    - **Terminal 1 (Backend):**
      ```bash
      npm run server
      ```
    - **Terminal 2 (Frontend):**
      ```bash
      npm run dev
      ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MS1317)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MS1317)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
