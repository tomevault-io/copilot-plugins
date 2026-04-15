## ielts-mate-fe

> A modern web application to help users prepare for the IELTS exam.

# IELTS Mate Web Application

A modern web application to help users prepare for the IELTS exam.  
Built for usability, accessibility, and effective IELTS skill-building.

---

## 1. Project Overview

- **Goal:** Provide comprehensive IELTS preparation, focused on Reading and Listening.
- **Key Features:** Practice tests, vocabulary lists, skill-specific tips, and a user-friendly interface.
- **Tech Stack:**
  - **Frontend:** React, TypeScript, ShadcnUI, Tailwind CSS
  - **Styling:** Tailwind CSS with a custom OKLCH-based color palette

---

## 2. Code Architecture

- Components are **modular** and separated by concern.
- **Easy to maintain:** Clear file/folder structure, readable code.
- **Responsibility separation:** Each component handles a distinct UI feature or logic unit.

---

## 3. UI & UX Guidelines

### 3.1. General Principles

- **Use Shadcn UI** for all components
- **Fully responsive:** Works on all devices
- **Accessible:** Keyboard navigation, strong color contrast
- **User-friendly:** Clear navigation and consistent layout

### 3.2. Brand Color Palette

- Color Usage Reference:

  - App background: Use bg-white
  - Card/Section background: Use bg-white
  - Primary text: tekhelet-400 or medium-slate-blue-400
  - Secondary text: tekhelet-500 or medium-slate-blue-500
  - Links/Highlight: medium-slate-blue-300 or tekhelet-300
  - Primary button: selective-yellow-300 (text: bg-white)
  - Warning/Alert: persimmon-300 or Tailwind’s default red scale.
  - Border/Divider: tekhelet-900
  - Success/Correct: Use Tailwind’s default green scale.

> Never use faded text or inaccessible contrast. Always meet accessibility standards.

### 3.3. Styling & Component Rules

- Tailwind Only: Use Tailwind classes for all color and effect styling.
- No inline CSS for colors unless absolutely necessary.
- When unsure of color, use the nearest semantic match from the palette.
- DO NOT add border lines unless specified. Use shadows for depth.

### 4. Summary Checklist

- Use ONLY the provided OKLCH color palette for all design.
- All styling should use Tailwind classes.
- Ensure accessibility and responsive design at every step.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vietqtran) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
