## rickyalves

> This repository contains a Next.js application that serves two primary purposes:

# Project: Ricky's Code Chronicles & Escola Sabatina Interativa

This repository contains a Next.js application that serves two primary purposes:
1.  **Ricky's Code Chronicles:** A personal portfolio and blog.
2.  **Escola Sabatina Interativa (Interactive Sabbath School):** A specialized interactive presentation application for Sabbath School (specifically focusing on Psalms) featuring real-time voting, reactions, and 3D visualizations.

## 🛠 Tech Stack

*   **Framework:** Next.js 14+ (App Router)
*   **Styling:** Tailwind CSS, Framer Motion
*   **3D Graphics:** React Three Fiber, Drei, Three.js
*   **Real-time Communication:** Socket.io (with a custom Express server for local dev)
*   **Backend:** Node.js / Express (for the custom interaction server)

## 🚀 Getting Started

### Prerequisites

*   Node.js installed
*   npm installed

### Running the Application (Local Development)

To fully utilize the interactive features (voting, reactions) locally, you need to run both the Next.js frontend and the custom Socket.io server.

1.  **Install Dependencies:**
    ```bash
    npm install
    ```

2.  **Start the Real-time Server (Terminal 1):**
    This server handles WebSocket connections for instant interactions.
    ```bash
    node server.js
    ```
    *   Runs on port `3001`.

3.  **Start the Next.js Frontend (Terminal 2):**
    ```bash
    npm run dev
    ```
    *   Runs on port `3000`.

### Accessing the App

*   **Portfolio Home:** `http://localhost:3000`
*   **Sabbath School Presentation:** `http://localhost:3000/salmos` (Press `F` for fullscreen)
*   **Mobile Participant Interface:** `http://localhost:3000/mobile-interface`

## 📂 Project Structure

*   **`/app`**: Next.js App Router pages.
    *   `layout.jsx`: Root layout including global styles.
    *   `page.jsx`: Main landing page (Portfolio).
    *   `salmos/`: The main presentation route for the Psalms lesson.
    *   `mobile-interface/`: The mobile-optimized page for audience participation.
    *   `api/`: API routes (including Vercel-compatible socket adaptations).
*   **`/components`**: Shared UI components.
    *   `app/components/`: Specific components for the App Router pages (e.g., `Sanctuary3D.jsx`, `QRCodeInteraction.jsx`).
*   **`/public`**: Static assets (fonts, images, 3D models).
*   **`server.js`**: Custom Express server entry point for Socket.io implementation.
*   **`DEPLOY_GUIDE.md`**: Instructions for deploying to Vercel.
*   **`README-ESCOLA-SABATINA.md`**: Specific documentation for the Sabbath School app.

## 🎮 Interactive Features (Sabbath School)

*   **Presenter View:** Controls slides, polls, and displays real-time stats.
*   **Participant View (Mobile):** accessible via QR code, allows users to vote in polls and send emoji reactions (Amen, Hallelujah, etc.).
*   **3D Sanctuary:** A 3D interactive visual component.

## 🚢 Deployment

The project is configured for deployment on **Vercel**.
*   **Note:** The Vercel environment uses a polling/serverless adaptation for "real-time" features as it does not support persistent WebSocket servers like `server.js` natively.
*   See `DEPLOY_GUIDE.md` for detailed production setup.

## 📝 Conventions

*   **Styling:** Use Tailwind utility classes.
*   **Components:** React functional components.
*   **Animations:** Use Framer Motion for UI transitions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/henriquesuper) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
