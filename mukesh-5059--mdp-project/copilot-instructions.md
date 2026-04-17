## mdp-project

> This is a [Next.js](https://nextjs.org) project that uses `three.js` and `cannon-es` to create a 3D train simulation. The simulation visualizes the pressure exerted by the train's wheels on the track through a heatmap effect.

# Project Overview

This is a [Next.js](https://nextjs.org) project that uses `three.js` and `cannon-es` to create a 3D train simulation. The simulation visualizes the pressure exerted by the train's wheels on the track through a heatmap effect.

The main components of the application are:

-   `app/page.tsx`: The main entry point of the application, which loads the `Renderer` component.
-   `app/components/Renderer.tsx`: The core of the application, where the `three.js` scene, `cannon-es` physics world, and the train/track are created and animated.
-   `app/components/Train.tsx`: Defines the creation of a single train compartment, including its physics and visual representation.
-   `app/components/Track.tsx`: Defines the creation of a track segment, including its physics and a custom shader for the heatmap effect.
-   `app/components/MiniGraph.tsx`: A component that likely displays a graph of the pressure data.

# Building and Running

To get started with this project, follow these steps:

1.  **Install dependencies:**

    ```bash
    npm install
    ```

2.  **Run the development server:**

    ```bash
    npm run dev
    ```

    Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

## Available Scripts

-   `npm run dev`: Runs the application in development mode.
-   `npm run build`: Builds the application for production.
-   `npm run start`: Starts a production server.
-   `npm run lint`: Lints the codebase using ESLint.

# Development Conventions

-   The project is written in **TypeScript**.
-   It uses **Next.js** as the React framework.
-   **`three.js`** is used for 3D rendering.
-   **`cannon-es`** is used for physics simulation.
-   Styling is likely done using **Tailwind CSS**, as indicated by the `tailwind.config.mjs` and `postcss.config.mjs` files.
-   The code is formatted according to the rules in `.eslintrc.json`.
-   The project uses **ES modules** for both front-end and back-end code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mukesh-5059) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
