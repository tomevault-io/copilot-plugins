## rule-for-react

> Here’s a rewritten and clearer version of your **React Development Guidelines**, along with a few proposed additions to strengthen consistency, performance, and developer experience.

Here’s a rewritten and clearer version of your **React Development Guidelines**, along with a few proposed additions to strengthen consistency, performance, and developer experience.

---

# **React Development Guidelines**

## 🔌 API Types & Interfaces

* **Always use API types/interfaces**: Use defined types from the backend or API wherever possible for component props, state, and other logic. This ensures strong typing and reduces maintenance overhead.

## 🎨 Design & Theming

* **Use theme colors**: Never hardcode color values. Always use colors defined in the theme system.
* **Create new colors only when necessary**: Extend the theme thoughtfully—only add new colors if existing ones don’t fulfill the design need.
* **Minimize unnecessary effects**: Avoid excessive hover effects or visual noise. Simplicity over flash.
* **Design inspiration**: When in doubt, follow the minimal, modern aesthetics of **Vercel**, **Supabase**, or **Postman**.

## ⚛️ React-Specific Best Practices

* **Avoid unnecessary `useEffect`**: Only use `useEffect` when side effects (like API calls or subscriptions) are absolutely required.
* **Use `react-router` instead of `react-router-dom`**: Prefer `react-router` for cleaner routing logic and modern API usage. NEVER USE `react-router-dom`

## ✅ Code Quality

* **Consistent naming conventions**: Name files, components, hooks, and utilities clearly (e.g., `useAuthToken.ts`, `ProfileCard.tsx`).
* **No anonymous components**: Always use named components for better debugging and clarity.
* **Enforce linting and formatting**: Use strict ESLint rules and Prettier—no formatting exceptions.


## 🧠 Code Logic & Structure

* **Keep it DRY**: Reuse logic through custom hooks or shared utility functions to avoid repetition.
* **Split large components**: Break down overly complex components into smaller, modular parts—even across multiple files if necessary.
* **Use or create generic components**: Check if a component already exists before building a new one. Create shared components when patterns repeat.
* **Strict linting and formatting**: Enforce rules via ESLint and Prettier. No exceptions.
* **Group by feature, not type**: Organize files by feature domain (e.g., `/features/auth/`, `/features/dashboard/`) instead of by file type.

---
> Source: [jmerelnyc/elkar-tasks](https://github.com/jmerelnyc/elkar-tasks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
