## ravenous

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ravenous is a Yelp-like restaurant/business finder built with React 19 and Vite 7. It is in early development — currently has a `Business` component for displaying business listings (name, address, category, rating, review count) but the main `App.jsx` still contains the Vite starter template.

## Commands

- `npm run dev` — start dev server with HMR
- `npm run build` — production build to `dist/`
- `npm run lint` — ESLint (flat config, JS/JSX only)
- `npm run preview` — preview production build

## Tech Stack

- React 19 + Vite 7 (ES modules, JSX, no TypeScript)
- ESLint 9 flat config with react-hooks and react-refresh plugins
- No test framework configured yet
- No routing or state management libraries yet

## Project Structure

- `src/components/` — React components with co-located CSS files (e.g., `Business.jsx` + `Business.css`)
- `src/App.jsx` — root application component
- `src/main.jsx` — entry point, renders `<App />` into `#root` with StrictMode

## ESLint Notes

- `no-unused-vars` rule ignores variables starting with uppercase or underscore (`varsIgnorePattern: '^[A-Z_]'`)

---
> Source: [CondorCommodore/ravenous](https://github.com/CondorCommodore/ravenous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
