## iptv-web

> Welcome to the **IPTV Docs** workspace. This document outlines coding standards, directory conventions, and design requirements for AI agents working in this repository.

# AI Agent Instructions for IPTV Docs Project

Welcome to the **IPTV Docs** workspace. This document outlines coding standards, directory conventions, and design requirements for AI agents working in this repository.

## 📌 Project Overview
This project is a modern, high-performance web documentation hub and API playlist service built with Next.js. It distributes parsed JSON playlists to clients, tracks viewer sessions, and hosts application download routes.

## 🛠️ Technology Stack
* **Framework**: Next.js 16.x (App Router)
* **Library**: React 19
* **Language**: TypeScript (Strict Mode)
* **Styling**: TailwindCSS v4
* **Animations**: Framer Motion (`motion/react`)
* **Icons**: `lucide-react`, `react-icons`

## 📂 Core Conventions & Rules

### 1. Platform & Rebranding Guidelines
* **Name**: The application and project must always be referred to as `IPTV Docs`.
* **Platform Scope**: macOS download listing is disabled. Do not generate layout components, download actions, or guides for macOS. Only Windows and Linux builds are supported.
* **Portable Preferences**: Windows Setup `.exe` is an installer (non-portable). The only portable package format for Windows is the `.zip` archive. On Linux, AppImage is portable. Ensure guides separate these concepts clearly.

### 2. Styling & Design (WCAG AA Compliance)
* **Theme**: Deep dark backgrounds (`#070414`) with glassmorphic cards (`bg-slate-900/40 backdrop-blur-xl border border-white/15`).
* **Contrast Ratios**: Do not use extremely low contrast styles like `border-white/5` or `text-zinc-600`/`text-zinc-500` directly on dark panels. Use at least `border-white/15` and `text-zinc-400` / `text-zinc-300` to pass accessibility checks.
* **Animations**: Always implement subtle spring-fade staggered page transitions with Framer Motion on client page entry.

### 3. API Routes & Stats Tracking
* **Playlists**: Located in `app/data/*.json`. Accessed via `/playlist/[filename]` dynamically or `/available_playlist.json`.
* **Session Stats**: The tracking endpoint is `/api/iptv/stats`.
  * Accepts `playingNow` payload with complete channel attributes (logo, name, url, referer, origin, group, useProxy).
  * In-memory session tracking groups viewers by lowercase channel name. Returns the top 10 channels with their current active viewer count.

---
> Source: [SHAJON-404/iptv-web](https://github.com/SHAJON-404/iptv-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
