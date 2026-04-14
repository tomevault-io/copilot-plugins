## portfolio

> **Objective:** Engineer a high-performance, visually distinct portfolio website tailored for competitive software engineering internships and Ph.D. program applications.

# Master Architecture Plan: Interactive Academic & Professional Portfolio

## 1. Project Overview

**Objective:** Engineer a high-performance, visually distinct portfolio website tailored for competitive software engineering internships and Ph.D. program applications.
**Core Identity:** The site will heavily emphasize the intersection of computing, physics, and information theory through interactive, browser-based mathematical animations, while cleanly documenting research in machine learning and malware analysis.

## 2. Technical Stack (Current)

- **Framework:** Astro (Static Site Generation for zero-JS initial load)
- **Language:** TypeScript (Frontend logic) / Python (Local data generation)
- **Styling:** Tailwind CSS (Utility-first, dark-mode aesthetic)
- **Animations:** p5.js (HTML5 Canvas integration)
- **Content Management:** Local Markdown (`.md`) via Astro Content Collections
- **Hosting/Deployment:** Firebase Hosting (Global CDN, automated CI/CD via GitHub)

## 3. Development Phases

### Phase 1: Foundation & Deployment Pipeline (Days 1-2)

**Goal:** Establish the skeleton, routing, and live deployment loop.

- [ ] Initialize new Astro project with Tailwind CSS integration.
- [ ] Set up global dark-mode color palette and typography (Inter / Fira Code).
- [ ] Create core page routes: `index.astro`, `about.astro`, `projects.astro`, `experience.astro`.
- [ ] Configure Astro Content Collections schema for blog posts/research write-ups.
- [ ] Initialize Firebase Hosting and connect to GitHub repository for automated deployments on push.

### Phase 2: Visual Engineering & Canvas Integration (Days 3-7)

**Goal:** Build the interactive mathematical components that define the site's aesthetic.

- **Component A: Conway's Game of Life Background**
  - [ ] Create an isolated `<canvas>` Astro component.
  - [ ] Write TypeScript + p5.js logic for cellular automata grid.
  - [ ] Implement subtle dark-mode styling (low opacity cells) and mouse hover interactivity (seeding cells).
- **Component B: Epicycle Fourier Series Portrait**
  - [ ] Write local Python script (`numpy`, OpenCV) to extract contour path from a personal portrait.
  - [ ] Compute complex Fourier coefficients and export to `portrait-data.json`.
  - [ ] Write TypeScript + p5.js frontend component to parse JSON and animate the drawing using rotating epicycles.

### Phase 3: Content Structuring (Days 8-10)

**Goal:** Populate the site with academic and professional achievements.

- **Experience Section:**
  - [ ] Document Teaching Assistant responsibilities for CS 216.
  - [ ] Detail the UC Irvine remote research assistant internship.
- **Projects & Research Section:**
  - [ ] Draft deep-dive Markdown article on Malware Analysis.
  - [ ] Detail the architecture of custom hybrid deep learning models (CNNs and LSTMs).
  - [ ] Link to respective GitHub repositories and live code.
- **Skills & Resume:**
  - [ ] Create a scannable technical stack grid (Python, Golang, TS, ML Libraries).
  - [ ] Embed a downloadable, up-to-date PDF resume.

### Phase 4: Future Scalability (Post-Launch Roadmap)

**Goal:** Expand capabilities without rewriting the core architecture.

- **Advanced WebGL Integrations:** Integrate Three.js components on dedicated pages (e.g., interactive 3D spacetime grids or topological knots) to further showcase physics concepts.
- **Headless CMS Integration:** Swap local Markdown files for the Notion API, allowing for seamless blog publishing directly from a private Notion workspace.
- **Live ML Inference API:** Deploy a Python/TensorFlow Docker container to Google Cloud Run to process user-uploaded files through the CNN-LSTM malware model, with the Astro frontend pinging the API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/n-oli)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/n-oli)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
