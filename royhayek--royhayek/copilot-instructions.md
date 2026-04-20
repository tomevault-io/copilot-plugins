## royhayek

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio website for Roy el Hayek, a frontend developer. It's built with Next.js 14, React 18, and Tailwind CSS, showcasing projects and professional experience.

## Architecture

- **Framework**: Next.js 14 with App Router
- **Styling**: Tailwind CSS with dark mode support (class-based)
- **Icons**: React Icons library
- **Structure**:
  - `/src/app/` - Next.js App Router pages (layout.js, page.js)
  - `/src/components/` - Reusable React components (Layout, Navbar, Footer)
  - `/public/` - Static assets (images, icons)

## Key Components

- **Layout Component** (`src/components/Layout.jsx`): Main layout wrapper with dark mode state management using React useState
- **Home Page** (`src/app/page.js`): Portfolio showcase with project grid, social links, and responsive design
- **Dark Mode**: Implemented using Tailwind's class-based dark mode, managed in Layout component

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Lint code
npm run lint
```

## Configuration

- **Tailwind Config**: Custom font family "burtons", dark mode enabled, custom border widths
- **Next.js Config**: Default configuration with no custom settings
- **Fonts**: Uses Inter font from next/font/google

## Project Structure

The portfolio data is stored as an array in the main page component (`src/app/page.js:24-97`) containing project information including images, descriptions, and external links. Images are stored in the `/public/` directory and imported statically.

## Styling Notes

- Uses Tailwind utility classes throughout
- Dark mode classes prefixed with `dark:`
- Custom gradient backgrounds and responsive design
- Custom font "burtons" available in theme extension

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/royhayek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
