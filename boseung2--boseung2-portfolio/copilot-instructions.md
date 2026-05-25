## boseung2-portfolio

> This project is a personal academic website for 정보승 (Boseung Jung), built using the Hugo Blox framework. It serves as a comprehensive online CV, showcasing his skills, experience, education, and projects. The website is designed to be a professional and modern representation of his academic and professional profile.

# GEMINI Project Analysis

## Project Overview

This project is a personal academic website for 정보승 (Boseung Jung), built using the Hugo Blox framework. It serves as a comprehensive online CV, showcasing his skills, experience, education, and projects. The website is designed to be a professional and modern representation of his academic and professional profile.

The project is configured to be built and deployed using Netlify, as indicated by the `netlify.toml` file. The content is written in Markdown, and the site's structure and appearance are defined through Hugo's templating and configuration files.

## Building and Running

To work with this project locally, you need to have Hugo, Node.js, and pnpm installed.

**1. Install Dependencies:**

```bash
pnpm install
```

This command will install the necessary Node.js dependencies, including Tailwind CSS.

**2. Run the Development Server:**

```bash
hugo server
```

This command will start a local development server, and you can view the website at `http://localhost:1313/`. The site will automatically reload when you make changes to the content or configuration.

**3. Build for Production:**

```bash
hugo --minify
```

This command will generate the static website in the `public` directory. This is the version of the site that gets deployed.

## Development Conventions

*   **Content Management:** All website content is managed in the `content/` directory. The structure of the site is defined by the subdirectories and Markdown files within this directory.
*   **Author Information:** The primary author's information is located in `content/authors/admin/_index.md`. This file contains the author's biography, social links, interests, education, and work experience.
*   **Configuration:** The main configuration for the website is split across several files:
    *   `hugoblox.yaml`: Specifies the Hugo Blox template and version.
    *   `config/_default/hugo.yaml`: The main Hugo configuration file.
    *   `config/_default/params.yaml`: Contains site-wide parameters for appearance, SEO, header, footer, and other features.
    *   `config/_default/menus.yaml`: Defines the website's navigation menus.
*   **Styling:** The website uses Tailwind CSS for styling. The Tailwind configuration is likely located in the project's root directory.
*   **Deployment:** The website is deployed using Netlify. The `netlify.toml` file defines the build commands and environment variables for the deployment process.

---
> Source: [boseung2/boseung2_portfolio](https://github.com/boseung2/boseung2_portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
