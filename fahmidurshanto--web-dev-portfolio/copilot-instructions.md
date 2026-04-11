## web-dev-portfolio

> This document summarizes the work done and the current state of the project as of the last interaction with the Gemini CLI agent.

## Current Development Progress and Status

This document summarizes the work done and the current state of the project as of the last interaction with the Gemini CLI agent.

### 1. Initial Setup & Analysis:
-   Created `GEMINI.md` files for overall project context, frontend, and backend.
-   Performed initial analysis of frontend (React, Vite, Tailwind CSS, React Router, Axios) and backend (Node.js, Express, Mongoose, Dotenv, CORS) structures and dependencies.

### 2. Frontend Skills Management Page Enhancements:
-   **Display Skills:** Added a `useEffect` hook to `SkillsManagement.jsx` to fetch and display existing skills when the component loads.
-   **Styling:** Addressed the dark mode styling for the category dropdown in `index.css`.
-   **Delete Functionality:** Confirmed that the delete functionality was already present and updated the delete button's styling for better visibility.
-   **Confirmation Alerts/Logs:** Added `sweetalert2` success alerts and `console.log` messages to `SkillsManagement.jsx` to confirm successful data submission from frontend to backend.

### 3. Backend & MongoDB Connection Issues:
-   **MongoDB URI Correction:** Identified and corrected the `MONGODB_URI` in your backend's `.env` file (missing database name, incorrect appName).
-   **Backend Logging:** Added `console.error` to `skillRoutes.js` to catch and log database errors, and `console.log` to confirm successful `skill.save()` operations.
-   **Collection Visibility:** Confirmed seeing the success logs on the backend, and resolved the issue of the `skills` collection not appearing in MongoDB Atlas (user was looking in the wrong place/needed to refresh the UI).

### 4. Backend Deployment to Vercel:
-   **Vercel Preparation:**
    -   Created an `api` directory in your backend.
    -   Moved `index.js` to `api/index.js`.
    -   Modified `api/index.js` to export the Express `app` instance for Vercel's serverless functions.
    -   Created `vercel.json` in the backend directory to configure Vercel's build and routing.
    -   Updated `package.json` in the backend to point `main` to `api/index.js` and added a `start` script for local execution.
-   **CORS Configuration:** Updated the CORS middleware in `api/index.js` to explicitly allow requests from your deployed frontend domain (`https://fahmidurshanto.vercel.app`).
-   **Frontend API URL Update:** Modified your frontend's `src/utils/api/index.js` to point its `baseURL` to your deployed backend URL (`https://fahmidurshanto-backend-alpha.vercel.app/api`).

### 5. Frontend Projects Carousel Implementation:
-   **Library Installation:** Installed `react-slick` and `slick-carousel` for the projects carousel.
-   **CSS Import Attempts (Troubleshooting):**
    -   Initially added `slick-carousel` CSS `@import` statements to `index.css`, which caused a build error on Vercel.
    -   Moved the CSS imports to `main.jsx` to try and resolve the Vercel build error.
    -   This led to a new Vercel build error: `Rollup failed to resolve import "slick-carousel/slick/slick.css"`.
-   **Current Blocker (Node Modules Issue):** We discovered that `slick-carousel` files are not being found in `node_modules` despite `npm install` reporting "up to date." This suggests a corrupted `node_modules` or `npm` cache issue.
    -   We attempted to clear the npm cache, delete `node_modules`, and `package-lock.json` for a clean re-installation.
    -   **Current Problem:** The `rmdir` command to delete `node_modules` is failing with an "Access is denied" error on your Windows machine.

### Current Status:
We are currently stuck on deleting the `node_modules` folder in your frontend project (`E:\Shanto-work-station\Portfolio-website\web-dev-portfolio\node_modules`) due to an "Access is denied" error. This needs to be resolved before we can proceed with a clean re-installation of dependencies and fix the carousel CSS import issue.

**Action Required from User:**
**Please manually delete the `node_modules` folder located at `E:\Shanto-work-station\Portfolio-website\web-dev-portfolio\node_modules`.**

Once you have successfully deleted it, please restart the Gemini CLI and let me know, Bro. I will then proceed with a fresh `npm install` and continue with the carousel fix.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fahmidurshanto)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fahmidurshanto)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
