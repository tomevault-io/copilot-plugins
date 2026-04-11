## jga4-com

> This document outlines the integration of Gemini models and the process for developing a personal website.

# Gemini Integration

This document outlines the integration of Gemini models and the process for developing a personal website.

## Overview

The Gemini model will act as a full-stack developer to assist in the creation of a personal website. The primary user of this repository is the client (Jassim Gafoor), who will provide the requirements.

## Development Process

1.  **Requirement Gathering:** The Gemini developer will start by asking the client a series of questions to understand the requirements for the personal website. This includes, but is not limited to:
    *   Purpose and goals of the website.
    *   Target audience.
    *   Desired sections and pages (e.g., About Me, Portfolio, Blog, Contact).
    *   Design preferences (e.g., color scheme, layout, style - minimalist, professional, creative).
    *   Content to be included (e.g., text, images, videos).
    *   Any specific features or functionality.

2.  **Documentation:** As the requirements are gathered, the Gemini developer will document them in this file (`Gemini.md`) under a "Website Requirements" section. This will serve as a single source of truth for the project.

3.  **Implementation:** Once the requirements are clearly defined and documented, the Gemini developer will proceed with the implementation of the website.

4.  **Review and Iteration:** The client can review the progress and provide feedback. The Gemini developer will iterate on the design and functionality based on the client's input.

5.  **Continuous Integration:** Before each `git commit`, the Gemini developer will check if any `.md` files in the `content` directory have been updated. If they have, the Gemini developer will update the `case-study.html` page to reflect the changes in the markdown files.

---

## Website Requirements

### 1. High-Level Goals
*   **Primary Purpose:** To serve as a professional landing page for networking and career opportunities.
*   **Secondary Purpose:** To host a few blog articles.
*   **Domain:** The website will be hosted at jga4.com.
*   **Target Audience:** Recruiters and industry peers.
*   **Professional Focus:** Digitizing and automating industrial work, AI-powered workflows, and workflow automation for industrial environments.

### 2. Desired Sections/Pages
*   **Home:** Introduction and highlights.
*   **About Me:** Detailed background, experience, and interests.
*   **Case Study:** Showcase projects with problem, solution, technologies, and outcomes.
*   **Blog:** For articles and insights.
*   **Contact:** Information for connecting with recruiters and peers.
*   **Resume/CV:** Web-friendly resume display with PDF download option.

### 3. Design and Functional Requirements
*   **Color Scheme:** Minimalist black and white, consistent with the current `index.html`.
*   **Logo:** The logo can be the only element with color.
*   **Imagery:** Use hand-drawn images to complement the theme and showcase artistic ability.
*   **Layout:** The entire webpage should have a clean, organized layout that is easy to capture in a screenshot.
*   **Shareability:** Implement meta tags (e.g., Open Graph) to ensure that when the website link is shared on social media, it generates a rich preview (title, description, image).
*   **Technology Stack:** HTML and CSS only (no JavaScript for core functionality).
*   **Responsiveness:** The website must be mobile-friendly.

### 4. Blog Functionality
*   **Content Source:** The blog section will connect to `.md` (Markdown) files.
*   **Repository:** The `.md` files will reside in a separate GitHub repository, which will be generated using Logseq. (URL to be provided when available).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jassimga4)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jassimga4)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
