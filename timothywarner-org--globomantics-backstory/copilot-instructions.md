## globomantics-backstory

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Globomantics Backstory repository - a collection of reusable educational assets themed around the fictional Globomantics Robotics company. The repository contains static assets (HTML/CSS/JS), images, datasets, and documentation designed for drop-in use in technical training and Pluralsight courses.

**Note**: This is primarily a static content repository, not a buildable application. There are no build scripts, test suites, or package managers to run.

## Repository Structure

```
/
├── apps/                    # Self-contained demo applications (planned restructure)
├── account-log-in/         # Login page mockups and implementations
├── account-sign-up/        # Sign-up page mockups  
├── dashboards/             # Analytics dashboard with Chart.js visualizations
├── datasets/               # CSV datasets for demos (HR, Financial, Marketing, etc.)
├── globomantics-asset-bundle/  # Core brand assets
│   ├── logos/              # SVG and PNG logos in various colors
│   ├── fonts/              # Open Sans font files
│   ├── color-palette/      # ASE color swatches
│   └── web-pages/          # Complete website templates
├── brand-identity/         # Brand guidelines and company backstory
├── 80s-items/             # Retro-themed images
├── green/, red/           # Status indicator icons
└── [various other assets]
```

## Key Information

### Brand Colors
- **Globo Blue**: #0066CC
- **Tech Black**: #1A1A1A  
- **Innovation Orange**: #FF6B35
- **Success Green**: #28A745
- **Electric Purple**: #7B2CBF
- **Data Cyan**: #00D4FF

### Typography
Primary font: Open Sans (included in `/globomantics-asset-bundle/fonts/`)

## Usage Guidelines

### Adding New Assets
1. Follow the self-contained app structure when adding new demos
2. Place all related assets (images, CSS, JS) within the same directory
3. Use relative paths for all asset references
4. Include a local README.md for complex demos

### Working with Existing Content
- Each top-level folder is designed to be self-contained and "pluckable"
- Copy entire folders to maintain asset dependencies
- The `/dashboards/` folder contains a working Chart.js analytics dashboard
- Dataset files in `/datasets/` are categorized by use case (HR, Financial, Marketing, etc.)

## Repository Restructure (In Progress)

The repository is being restructured to improve organization (see `REPO-RESTRUCTURE-PLAN.md`):
- Moving to an `/apps/` directory structure for all self-contained demos
- Better separation of code, assets, and documentation
- Enhanced portability for educational use

## Common Tasks

### Viewing HTML Demos
Simply open any HTML file directly in a browser. No build process required.

### Using Brand Assets
1. Logos are in `/globomantics-asset-bundle/logos/`
2. Use SVG versions when possible for scalability
3. Follow brand guidelines in `/brand-identity/brand-guidelines.md`

### Working with Datasets
CSV files in `/datasets/` are organized by category and ready for direct import into demos or data visualization tools.

## Important Notes

- This repository is designed for educational purposes
- All content is themed around the fictional Globomantics Robotics company
- Assets are meant to be copied and integrated into course materials
- No sensitive or real company data is included

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/timothywarner-org)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/timothywarner-org)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
