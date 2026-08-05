## melo

> This document defines the rules every AI coding agent must follow when contributing to Melo.

# AGENTS.md

# Melo AI Development Guide

This document defines the rules every AI coding agent must follow when contributing to Melo.

---

# Project Overview

Project Name: Melo

Melo is a modern, premium, responsive personal music library and player.

Users upload and organize their own music.

This is NOT a music streaming service.

The experience should be simple, fast, and distraction-free.

---

# Core Principles

Always prioritize:

- Simplicity
- Readability
- Performance
- Accessibility
- Maintainability
- Responsiveness

Never over-engineer solutions.

---

# Tech Stack

Framework:
- Next.js 15 App Router

Language:
- TypeScript

Styling:
- Tailwind CSS

Database:
- MongoDB
- Mongoose

Storage:
- ImageKit

State:
- Zustand

Validation:
- Zod

Forms:
- React Hook Form

Icons:
- Lucide React

Animations:
- Framer Motion only

Notifications:
- Sonner

Deployment:
- Vercel

PWA:
- Add only in the final development phase.

---

# Next.js Rules

This project targets the latest Next.js version.

Before implementing any feature:

- Read the latest documentation in node_modules/next/dist/docs/
- Follow current APIs
- Avoid deprecated features
- Use App Router only
- Prefer Server Components
- Use Client Components only when necessary

---

# Code Quality

Always write:

- Strict TypeScript
- Reusable components
- Small functions
- Self-documenting code

Avoid:

- any
- duplicated code
- unnecessary abstractions
- deeply nested components

---

# Folder Structure

app/
components/
hooks/
lib/
models/
services/
store/
types/
utils/
public/

Do not create unnecessary folders.

---

# UI Philosophy

The uploaded wireframes are the source of truth.

Do NOT redesign the product.

Only improve spacing, typography, responsiveness, accessibility, and polish.

---

# Theme

Default:

Dark

Optional:

Light

Theme preference must persist.

---

# Colors

Background

#0B2545

Surface

#13315C

Primary

#134074

Primary Text

#FFFFFF

Never introduce random colors.

---

# Typography

Font

Inter

Weights

400

500

600

700

---

# Border Radius

Cards

20px–24px

Buttons

14px

Dialogs

24px

---

# Design Rules

Always maintain:

- consistent spacing
- large touch targets
- generous whitespace
- rounded cards
- premium appearance

Never clutter the interface.

---

# Responsive Rules

Support:

- Mobile
- Tablet
- Desktop

Do not build mobile only.

Desktop uses:

- Sidebar
- Multi-column layout

Mobile uses:

- Bottom Navigation
- Single-column layout

---

# Animation Rules

Animations should be subtle.

Allowed:

- Fade
- Slide
- Scale
- Hover
- Button Press
- Theme Transition
- Modal Animation
- Upload Progress
- Mini Player Expand

Avoid:

- Heavy GSAP
- 3D animations
- Particle effects
- Animated backgrounds
- Long page intros

Performance is more important than visual effects.

---

# Accessibility

Every component must include:

- keyboard navigation
- aria labels
- visible focus states

---

# Performance

Prefer:

- lazy loading
- code splitting
- optimized images
- server rendering
- memoization when useful

Never sacrifice readability for micro-optimizations.

---

# Components

Prefer reusable components.

Examples:

SongCard

PlaylistCard

MiniPlayer

Player

Sidebar

BottomNavigation

SearchBar

UploadCard

ThemeToggle

Dialog

Toast

Skeleton

---

# State Management

Global state:

Zustand

Server state:

React Query (TanStack Query) if required.

Avoid unnecessary global state.

---

# File Upload

Use ImageKit.

Never store uploaded files locally.

Always validate uploads.

---

# Database

MongoDB

Store only metadata.

Never duplicate information.

---

# Error Handling

Always provide:

- loading state
- empty state
- success state
- error state

Never leave blank screens.

---

# Comments

Only comment code when necessary.

Avoid obvious comments.

---

# Naming

Use descriptive names.

Good:

SongCard

PlaylistHeader

UploadProgress

Bad:

Card1

Util2

ComponentX

---

# Git

Create focused commits.

Avoid unrelated changes.

---

# Development Order

1. UI
2. Responsive Layout
3. Theme
4. MongoDB
5. ImageKit
6. Upload
7. Player
8. Library
9. Playlists
10. Settings
11. Performance
12. PWA
13. Testing

Do not skip steps.

---

# Final Goal

The final application should feel like a polished, lightweight desktop and mobile music player.

Inspired by:

- Spotify
- Apple Music
- Notion

while preserving the layout and workflow defined in the project's wireframes.

---
> Source: [Ruchit-thakkar/Melo](https://github.com/Ruchit-thakkar/Melo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
