## hugo-genealogy-blog

> This document provides a high-level overview of the `hugo-genealogy-blog` project, outlining its structure, technologies, and operational procedures.


# Gemini Code Assistant Project Overview

This document provides a high-level overview of the `hugo-genealogy-blog` project, outlining its structure, technologies, and operational procedures.

## Project Description

This is a personal genealogy blog created with the [Hugo](https://gohugo.io/) static site generator. It uses the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. The site is deployed on Netlify.

## Key Technologies

- **Static Site Generator:** Hugo v0.142.0
- **Theme:** PaperMod
- **Hosting:** Netlify
- **Code Formatting:** Prettier with `prettier-plugin-go-template` for Hugo templates.

## Project Structure

- `content/`: Contains the Markdown content for the blog posts and pages.
- `static/`: Holds static assets like images, CSS, and JavaScript files.
- `layouts/`: Contains custom layouts and shortcodes to override theme defaults.
- `config.yml`: The main Hugo configuration file.
- `netlify.toml`: Netlify deployment and build settings.
- `package.json`: Node.js dependencies, primarily for development tooling like Prettier.

## Getting Started

### Prerequisites

- [Hugo](https://gohugo.io/getting-started/installing/) (version 0.142.0 is recommended)
- [Node.js](https://nodejs.org/en/) and [pnpm](https://pnpm.io/installation)

### Installation

1. Clone the repository.
2. Install the Git submodule for the theme:
   ```bash
   git submodule update --init --recursive
   ```
3. Install Node.js dependencies:
   ```bash
   pnpm install
   ```

### Running the Development Server

To build the site and run a local server with live reloading, use the development command found in `netlify.toml`:

```bash
hugo -b http://127.0.0.1:8888/ -w --buildDrafts -F
```

This will start a server at `http://127.0.0.1:8888/`.

### Building for Production

To create a production build of the site, use the build command from `netlify.toml`:

```bash
hugo --gc --minify
```

The output will be generated in the `public/` directory.

## Content Creation

New content is added to the `content/posts/` directory. Each post should be in its own subdirectory and have an `index.md` file.

## Code Style

This project uses [Prettier](https://prettier.io/) to format Go HTML template files. To format your code, run:

```bash
pnpm prettier --write .
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miams) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
