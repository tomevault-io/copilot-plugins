## koushilmankali-in

> This project is a personal portfolio website built with Next.js and React. It showcases the developer's projects, skills, and contact information. The site is designed to be a comprehensive showcase of their work, including detailed pages for each project.

# Project: Personal Portfolio

## Overview

This project is a personal portfolio website built with Next.js and React. It showcases the developer's projects, skills, and contact information. The site is designed to be a comprehensive showcase of their work, including detailed pages for each project.

The portfolio leverages Next.js for server-side rendering and static site generation, providing good performance and SEO. Project details are managed as Markdown files within the `posts/` directory, which are parsed using `gray-matter`. This allows for easy content management of the projects showcased.

## Technologies Used

-   **Framework:** Next.js
-   **Library:** React.js
-   **Styling:** CSS Modules
-   **Content:** Markdown (`react-markdown`, `gray-matter`)
-   **Server:** Custom Node.js/Express server (`server.js`)
-   **Linting:** ESLint

## Building and Running the Project

To get the project running locally, use the following commands:

1.  **Install Dependencies:**
    ```bash
    npm install
    ```

2.  **Run in Development Mode:**
    Starts the development server on `http://localhost:3000`.
    ```bash
    npm run dev
    ```

3.  **Build for Production:**
    Creates an optimized production build of the application.
    ```bash
    npm run build
    ```

4.  **Run in Production Mode:**
    Starts the production server. This should be run after building the project.
    ```bash
    npm start
    ```

## Development Conventions

-   **Component Structure:** Components are organized into their own folders within the `components/` directory. Each component folder contains the component's JavaScript file and its corresponding CSS module (e.g., `MyComponent/MyComponent.js` and `MyComponent/MyComponent.module.css`).
-   **Project Data:** Project information is primarily managed in two ways:
    1.  A hardcoded array of project objects within `components/ProjectsComponent/ProjectsComponent.js`. This is used to render the main project list.
    2.  Individual Markdown files in the `posts/` directory. Each file represents a detailed project page. `lib/post-util.js` contains the logic for reading and parsing these files.
-   **Routing:** The application uses the file-system based routing from Next.js, located in the `pages/` directory. Dynamic routes are used for individual post pages (e.g., `pages/posts/[postId].js`).
-   **Custom Server:** A custom Node.js server (`server.js`) is used, which is not standard for all Next.js projects. This suggests there might be custom server-side logic or routing, although the current implementation is basic.
-   **Sitemap:** A sitemap is generated automatically as a pre-build step using the `lib/sitemapcjs.js` script.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/koushil-mankali) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
