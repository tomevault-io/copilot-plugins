## aba-github-io

> This file serves as a high-signal architectural guide and reproducible development manual for future Gemini AI sessions and other developers working on this portfolio workspace.

# Astro Portfolio & CV Developer Guide (GEMINI.md)

This file serves as a high-signal architectural guide and reproducible development manual for future Gemini AI sessions and other developers working on this portfolio workspace.

---

## 🏗️ System Tech Stack

The portfolio is engineered as a lightning-fast, modern static site with a sleek, tech-focused control room aesthetic.

1. **Framework:** **Astro v6** (fully static outputs, optimized layout slots).
2. **Styling:** **Tailwind CSS v4** (integrated directly into the build pipeline via `@tailwindcss/vite` plugin inside `astro.config.mjs`).
3. **Typography:** **Inter** (sans-serif for readable copy) & **JetBrains Mono** (monospaced for metrics, scopes, and code blocks).
4. **Icons:** **Lucide Icons** (`@lucide/astro`).
5. **Typesetting:** **Typst** (inside the `CV/cv-kit/` directory) for clean, programmatic CV design.

---

## 📂 Project Architecture

```text
/
├── public/                  # Static assets (favicons, CV PDF resume)
│   ├── resume.pdf           # The compiled and copied Typst CV PDF
│   └── images/              # System/Scope trace images (g722, nameonchip)
├── src/
│   ├── layouts/
│   │   └── BaseLayout.astro # Base HTML, fonts, and dark slate gradient
│   ├── components/
│   │   ├── Projects.astro            # Featured projects metadata & lists
│   │   ├── Experience.astro          # Timeline of professional R&D experience
│   │   ├── Skills.astro              # Transparent technical competencies and workflows
│   │   ├── CodeShowcase.astro        # VS Code-style Shiki syntax highlighter
│   │   ├── ArchitectureDiagram.astro # High-fidelity semantic HTML sequence flows
│   │   └── SystemViewScope.astro     # Retro CRT oscilloscope tracing simulator
│   └── pages/
│       └── index.astro      # Core page assembling all blocks
├── flake.nix                # Reproducible Nix dev shell declaration
└── package.json             # NPM dependencies & scripts (Node.js >=22.12.0)
```

---

## ❄️ Reproducible Nix Development Environment

To guarantee complete parity across all compilation machines (eliminating the "works on my machine" problem), a unified Nix Flake environment is declared in the root.

### Dependencies Declared:
*   `nodejs_22` & `nodePackages.npm` (Website dev stack)
*   `typst` (CV typesetting compiler)

### Quick Start with Nix:
Ensure Nix is installed and experimental features are enabled (`flakes` and `nix-command`). Then, run:

```bash
nix develop
```

This will automatically drop you into a shell with Node 22, NPM, and Typst loaded on-demand.

---

## 🧞 Development & Compilation Commands

All commands below should be run from inside the active `nix develop` shell:

### 1. Web Page Development

```bash
# Install NPM dependencies
npm install

# Start local Astro live-reload dev server (default port 4321)
npm run dev

# Build the final optimized static production bundle to ./dist/
npm run build

# Preview the locally built static build
npm run preview
```

### 2. Typst CV Typesetting & Deployment

```bash
# Navigate to the CV directory
cd CV/cv-kit

# Compile the Typst source code into a PDF
typst compile abanoub-cv.typ abanoub-cv.pdf

# Watch the file for live compilation on edits
typst watch abanoub-cv.typ abanoub-cv.pdf

# Copy the compiled CV directly to the web public assets folder
cp abanoub-cv.pdf ../../public/resume.pdf
```

---

## 🚀 Key Achievements & Custom Components

*   **`SystemViewScope.astro`:** Implements a visual overlay mimicking a real-world Segger SystemView or hardware trace oscilloscope. Uses CSS linear-gradients to build a grid and a subtle CRT scanline effect over dynamic captures.
*   **ARM Cortex-M4 Secure FOTA Sandbox (New Project):** Added to the top of `Projects.astro` to showcase your local R&D sandbox (QEMU guest, mTLS Mosquitto broker, asyncio AWS IoT jobs emulator, and Nix Flake environments). Renders both the custom `flake.nix` environment file and an interactive TLS 1.3 sequence handshake.

---
> Source: [AbanoubN/aba.github.io](https://github.com/AbanoubN/aba.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
