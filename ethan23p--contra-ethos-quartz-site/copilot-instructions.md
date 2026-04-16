## contra-ethos-quartz-site

> This project is a digital garden built with [Quartz](https://quartz.jzhao.xyz/), a static site generator that transforms a set of Markdown files into a website. The entire site content is written and managed within [Logseq](https://logseq.com/). Quartz reads these Markdown files, processes them through a series of plugins, and generates the final static site, which is then hosted using GitHub Pages.

# Project Overview

This project is a digital garden built with [Quartz](https://quartz.jzhao.xyz/), a static site generator that transforms a set of Markdown files into a website. The entire site content is written and managed within [Logseq](https://logseq.com/). Quartz reads these Markdown files, processes them through a series of plugins, and generates the final static site, which is then hosted using GitHub Pages.

The project is configured to use a variety of plugins for processing Markdown, including support for Obsidian-flavored Markdown, syntax highlighting, and LaTeX. The visual theme and layout are customized in `quartz.config.ts` and `quartz.layout.ts`.

## Building and Running

To build the site and serve it locally, use the following command:

```bash
npx quartz build --serve
```

This will start a local server at `http://localhost:8080`.

## Gemini Development Guidelines

As an AI pair programmer, I will adhere to the following guidelines to ensure our collaboration is smooth, effective, and maintainable.

### 1. Leverage Local Documentation

This project includes the complete Quartz 4 documentation in the `docs/` directory. I will consult these local files first to understand Quartz's architecture, features, and conventions before seeking external information. This ensures my actions are grounded in the project's specific version and capabilities.

### 2. Non-Intrusive Modifications

Our primary goal is to customize and extend Quartz without making it difficult to update in the future.

*   **CSS Changes:** I will **never** edit `.css` files directly. All custom styling will be implemented by modifying `quartz/styles/custom.scss`. This file is the designated place for theme overrides and new styles, ensuring a clean separation from the core stylesheets.
*   **Core Components:** When modifications to core components are necessary, I will proceed with caution. I will prioritize creating new, custom components or using existing extension points (like plugins) over directly altering the files in `quartz/components` and `quartz/plugins`. This "update-aware" approach will make it easier to incorporate future Quartz updates.

### 3. Modification Strategy

I will follow a structured approach to making changes:

*   **Configuration First:** For broad, site-wide changes (e.g., theme, navigation, plugin settings), I will start by modifying `quartz.config.ts` and `quartz.layout.ts`.
*   **Source Code for Depth:** For more advanced customizations, such as creating new components or plugins, I will read the relevant source code within the `quartz/` directory to understand the underlying implementation details.

## Development Conventions

### Code Style

The project uses [Prettier](https://prettier.io/) for code formatting. To check for formatting issues, run:

```bash
npm run check
```

To automatically format the code, run:

```bash
npm run format
```

### Testing

The project uses `tsx` for running tests. To run the test suite, use the following command:

```bash
npm run test
```

### Content Workflow

The website's content is located in the `content` directory, which Quartz uses to generate the static site. However, the author, Ethan, uses Logseq to write and manage all content. Therefore, the `content` folder should be considered an artifact of the Logseq-to-Quartz pipeline. To avoid conflicts and ensure a smooth workflow, we will not edit the files within this directory directly during our pair programming sessions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ethan23p) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
