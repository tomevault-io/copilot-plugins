## landing

> This project is a static, single-page, responsive landing website for **Neurêkha Clinique**, a neuropsychology and cognitive rehabilitation clinic.

# GEMINI.md - Project Context

## Project Overview

This project is a static, single-page, responsive landing website for **Neurêkha Clinique**, a neuropsychology and cognitive rehabilitation clinic.

- **Technology:** The site is built with plain HTML. All styling (CSS) and functionality (JavaScript) are embedded directly within the `index.html` file for maximum simplicity and portability.
- **Architecture:** This is a flat, serverless project. There is no backend, database, or build process. It is designed to be hosted on any simple static web hosting service.
- **Assets:** All visual assets (logo, content images) are located in the `/images` directory.
- **Dependencies:** The only external dependency is Google Fonts (`Lora` and `Poppins`), which are linked in the `<head>` of the HTML.

## Building and Running

### Running Locally

There is no build step. To view the website, simply open the `index.html` file in any modern web browser.

### Deployment

To publish the website, upload the following to any static web hosting provider (like Netlify, Vercel, GitHub Pages, etc.):

1.  The `index.html` file.
2.  The entire `images/` folder.

## Development Conventions

- **File Structure:** The structure is simple: `index.html` at the root and all images in the `/images` directory.
- **Styling:** CSS is located within a `<style>` block in the `<head>` of `index.html`. It uses CSS variables for the color palette.
- **JavaScript:** A small amount of vanilla JavaScript for the mobile menu and scroll effects is located in a `<script>` block at the end of the `<body>`.
- **Dependencies:** The only external dependencies are Google Fonts, linked in the `<head>`.
- **Hero Section:** The hero section must be a full-width banner. **Do not use a split-screen layout.**

---
> Source: [neurekha/landing](https://github.com/neurekha/landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
