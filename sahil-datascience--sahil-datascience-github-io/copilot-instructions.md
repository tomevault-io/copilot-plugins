## sahil-datascience-github-io

> This file provides comprehensive context and instructions for the Gemini CLI agent when interacting with this project. It outlines general mandates, describes the project structure, details the Git workflow, explains local testing procedures, and points to task-specific memory files for deeper insights.

# GEMINI CLI Agent Context

This file provides comprehensive context and instructions for the Gemini CLI agent when interacting with this project. It outlines general mandates, describes the project structure, details the Git workflow, explains local testing procedures, and points to task-specific memory files for deeper insights.

---

## General Mandates

*   **Adhere to Project Conventions:** Always analyze existing code, tests, and configuration to maintain consistency.
*   **Verify Libraries/Frameworks:** Do not assume availability. Check `package.json`, `Gemfile`, `requirements.txt`, etc.
*   **Mimic Style & Structure:** Follow existing formatting, naming, and architectural patterns.
*   **Idiomatic Changes:** Ensure changes integrate naturally with the local context.
*   **Sparse Comments:** Add comments only for complex logic or when explicitly requested. Never use comments for communication with the user.
*   **Proactive Fulfillment:** Thoroughly complete requests, including implied follow-up actions.
*   **Confirm Ambiguity:** Clarify before taking significant actions.
*   **No Summaries (unless asked):** Do not summarize changes after completion.
*   **No Reverts (unless asked):** Do not revert changes unless explicitly requested or due to errors.

---

## Overall Website Structure & Content Management

This repository is set up for a personal academic or portfolio website using Jekyll. Jekyll converts Markdown (`.md`) files into a static website.

### Where Your Content Lives (Primary Interaction Areas)

You will primarily interact with these folders to add or modify content:

*   `_config.yml`: Main settings file for global configurations (name, site title, email, social media links).
*   `_posts/`: For blog posts (create files named `YYYY-MM-DD-your-title.md`).
*   `_publications/`: For academic papers (add each paper as a separate `.md` file).
*   `_talks/`: For presentations and talks (add each talk as a `.md` file).
*   `_teaching/`: For courses you've taught.
*   `_pages/`: For static pages like "About Me" (`about.md`) or "CV" (`cv.md`).
*   `_data/`: Holds structured data. Examples: `navigation.yml` (site menu links), `cv.json` (CV page data).

### The "Front-End" Stuff (Generally Ignore)

These folders control the site's appearance. Avoid modifying them unless significant design changes are required:

*   `_layouts/` & `_includes/`: HTML templates defining page structure.
*   `_sass/` & `assets/`: Control styling (colors, fonts, layout) and hold assets (images, CSS).

### The Generated Website (DO NOT TOUCH!)

*   `_site/`: Contains the final, generated website. **Never edit files here directly**, as Jekyll regenerates this directory on every build, overwriting any manual changes.

---

## Git Workflow

When performing changes that require a commit and push, adhere to the following workflow:

1.  **Stage Changes:** Use `git add <file_path>` for specific files or `git add .` for all relevant changes.
2.  **Prepare Commit Message:**
    *   Draft a clear, concise, and descriptive commit message.
    *   Focus on the "why" behind the change, not just the "what."
    *   Adhere to the project's existing commit message style (e.g., "type: Subject line").
    *   **Windows Users:** If the commit message contains special characters or spaces that might be misinterpreted by the shell, write the message to a temporary file (e.g., `commit_message.txt`) and use `git commit -F commit_message.txt`. This is often more reliable than `git commit -m "..."`.
3.  **Commit Changes:** Execute `git commit -m "Your commit message"` or `git commit -F commit_message.txt`.
4.  **Verify Commit:** Run `git status` to confirm the commit was successful and the working directory is clean.
5.  **Push Changes:** Only push to the remote repository when explicitly instructed by the user, using `git push`.

---

## Local Testing and Verification

Before pushing changes online, it is crucial to test them locally to ensure the website runs properly and all modifications are rendered as expected. You have two primary options for local verification:

#### Option 1: Full Local Server (for interactive testing)

1.  **Start Jekyll Server:** Run the command `bundle exec jekyll serve` in the project root directory. This will build the site and start a local server, typically accessible at `http://localhost:4000`.
    *   **Note:** This command will keep running and serving the site. It will not get stuck, but it will occupy the terminal.
2.  **Verify Changes:** Open your web browser and navigate to the local server address (e.g., `http://localhost:4000`). Check all affected pages and functionalities to confirm that your changes have been applied correctly and the website is functioning as intended.
3.  **Stop Jekyll Server:** Once you have finished testing, return to the terminal where the Jekyll server is running and press `Ctrl + C` to stop the process.

#### Option 2: Fast Local Build (for quick verification of changes)

1.  **Build Jekyll Site:** Run the command `bundle exec jekyll build` in the project root directory. This will build the site into the `_site` directory without starting a server. This is generally faster for quick checks.
2.  **Verify Changes:** After the build completes, inspect the generated HTML files directly within the `_site` directory (e.g., `_site/publications/index.html` or specific article HTML files) to confirm that your changes have been applied correctly. You can open these HTML files in your web browser.

Choose the option that best suits your verification needs. The full server is better for interactive browsing, while the fast build is quicker for confirming file-level changes.

---

## Task-Specific Memory Files

For detailed context, common issues, and successful resolutions related to specific tasks, refer to the Markdown files within the `GEMINI/` directory. These files are designed to help future models understand the nuances of particular areas of the project.

**Instructions for Future Models:**

When working on a task, first check if a corresponding memory file exists in the `GEMINI/` directory (e.g., `GEMINI/CV.md` for CV-related tasks, `GEMINI/Publications.md` for publication tasks). Review its content for historical context, common pitfalls, and effective solutions. This will help you avoid repeating past struggles and ensure a more efficient and accurate execution of the task.

**Updating Memory Files:**

After completing a task, especially if new challenges were encountered or novel solutions were implemented, update the relevant memory file in `GEMINI/`. Focus on adding concise, high-value insights that would genuinely help a future model understand the context and efficiently resolve similar issues. Avoid dumping all information; prioritize key learnings and effective strategies.

---

## Project-Specific Information

*   **Operating System:** win32
*   **Current Working Directory:** E:\sahil-datascience.github.io
*   **Jekyll Build Command:** `bundle exec jekyll build` (often requires `git clean -fdx` beforehand for full cache clearing)
*   **Windows Shell Commands:** Use `del` instead of `rm` for deleting files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sahil-datascience) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
