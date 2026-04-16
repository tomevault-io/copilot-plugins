## kuchida1981-github-io

> This is the source code for the personal blog "U-REI.com" (kuchida1981.github.io). It is a static site generated using [Hugo](https://gohugo.io/) with the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. The content is primarily in Japanese.

# Project Overview

This is the source code for the personal blog "U-REI.com" (kuchida1981.github.io). It is a static site generated using [Hugo](https://gohugo.io/) with the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme. The content is primarily in Japanese.

## Key Technologies
- **Hugo:** Static site generator (Version 0.145.0+ extended).
- **PaperMod:** The theme used for the blog.
- **Docker:** Used for consistent development environment.
- **GitHub Actions:** Handles the build and deployment process.

# Building and Running

## Local Development (Docker - Recommended)
To start the development server using Docker:
```bash
docker compose up -d
```
The site will be available at `http://localhost:1313/`.

To create a new post using Docker:
```bash
docker compose run --rm hugo new posts/YYYY/MM/slug.md
```

## Local Development (Hugo CLI)
If you have Hugo installed locally:
```bash
hugo server -D
```

To create a new post:
```bash
hugo new posts/slug.md
```

## Building for Production
The project is built automatically via GitHub Actions on push to `master`. To build locally:
```bash
hugo --gc --minify
```
Output is generated in the `docs/` directory.

# Development Conventions

## Language
- **Communication:** Japanese (`ja-JP`).
- **Content:** The blog content is written in Japanese.

## Agent Workflow (OpenSpec)
This project uses **OpenSpec** for managing changes and specifications.

**CRITICAL:** Always refer to `@/openspec/AGENTS.md` when:
- The user requests a plan, proposal, or spec change.
- The request involves new capabilities, breaking changes, or architectural shifts.
- The request is ambiguous and requires clarification.

## Directory Structure
- `config.yml`: Main Hugo configuration.
- `content/`: Markdown content for the site.
- `themes/papermod/`: The submodule for the PaperMod theme.
- `openspec/`: Contains project specifications and change proposals.
- `.github/workflows/`: CI/CD configurations.
- `docs/`: The build output directory (published via GitHub Pages).

## Deployment
Changes merged into the `master` branch are automatically built and deployed to GitHub Pages via the workflow defined in `.github/workflows/hugo.yaml`.

# Automated Features



## Daily Blog Post Generation

A GitHub Actions workflow (`.github/workflows/daily-post.yaml`) runs daily to draft a new blog post using Gemini.



**Requirements:**

- **GitHub Secrets**: `GEMINI_API_KEY` must be set in the repository secrets for the generation to work.



### Local Testing (ローカルでの動作確認)

To test the generation script locally:



1.  **Prepare Environment**: Create a `.env` file in the root directory.

    ```bash

    echo "GEMINI_API_KEY=your_api_key_here" > .env

    ```

2.  **Install Dependencies**:

    ```bash

    pip install -r scripts/requirements.txt

    ```

3.  **Run Script**:

    ```bash

    python scripts/generate_daily_post.py

    ```

    The generated post will be saved to `content/posts/YYYY-MM-DD-daily-news.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kuchida1981) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
