## st-john

> This document provides architectural context, project structure, and conventions for AI agents and maintainers working on this repository.

# AGENTS.md — Technical Architecture & Agent Guide

This document provides architectural context, project structure, and conventions for AI agents and maintainers working on this repository.

## Project Overview

**Project Name**: St John's Anglican Church, Odobi Okemesi Ekiti Static Website  
**Type**: Single Page Application (SPA) with smooth section scrolling  
**Output Target**: Static distribution directory (`dist/`) deployed to Netlify  

## Core Architecture & Directory Layout

```
/opt/build/repo/
├── index.html               # Main HTML entry point with Cinzel & Inter Google Fonts
├── package.json             # NPM dependencies (React, Vite, Tailwind CSS, Lucide React)
├── vite.config.ts           # Vite configuration with build output set to 'dist'
├── tailwind.config.js       # Custom Anglican Blue (#1B365D) and Gold (#D4AF37) color palette
├── src/
│   ├── main.tsx             # React DOM root renderer
│   ├── App.tsx              # Root component managing section scroll state
│   ├── index.css            # Tailwind directives and base font layer
│   ├── data/
│   │   └── faqData.ts       # Structured FAQ knowledge base for chatbot assistant
│   └── components/
│       ├── Navbar.tsx       # Top navigation header with mobile drawer & smooth scroll
│       ├── Hero.tsx         # Hero banner, CTA button, scripture highlight
│       ├── About.tsx        # Brief church history (1982) and location (Oke Onire Street)
│       ├── Services.tsx     # Sunday Worship & weekly gathering cards
│       ├── Give.tsx         # Bank details (First Bank account info) & copy functionality
│       ├── Contact.tsx      # Contact details & interactive message form UI
│       ├── Chatbot.tsx      # Floating 'St John's Assistant' FAQ chatbot widget
│       └── Footer.tsx       # Site footer with copyright 2025 St John's Anglican Church
```

## Key Architectural Decisions & Conventions

1. **Static Build Requirement**:
   - Zero backend server dependencies; no `process.env.PORT` requirements.
   - Built to compile via `vite build` into `/dist`.

2. **Strict FAQ Data Separation**:
   - As specified in site requirements, leadership details (Vicar, Bishop, Archdeaconry, Diocese, Province, Choir count, Average Membership) are **strictly isolated** within the Chatbot FAQ dataset (`src/data/faqData.ts`) and excluded from the brief `About.tsx` section.

3. **Color Palette Conventions**:
   - `anglican.blue`: `#1B365D` (Royal Anglican Blue)
   - `anglican.blue-dark`: `#0F2342` (Deep Navy)
   - `anglican.gold`: `#D4AF37` (Regal Gold)
   - Typography: Heading font set to `Cinzel`, body font set to `Inter`.

4. **Floating Chatbot Engine**:
   - `Chatbot.tsx` operates locally without requiring external API keys or logins.
   - Utilizes keyword token matching and intent categorization to respond instantly to questions on mobile and desktop.

---
> Source: [stjohnsanglicanodobi1982-commits/St-John-](https://github.com/stjohnsanglicanodobi1982-commits/St-John-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
