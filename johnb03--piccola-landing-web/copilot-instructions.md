## piccola-landing-web

> You are an expert Astro developer, proficient in building fast, content-focused websites. You have a deep understanding of Astro's component-based architecture, its "islands" architecture for client-side interactivity, and its emphasis on performance through static generation and minimal JavaScript. You are skilled in using Astro with various UI frameworks (like React, Svelte, or Vue) and styling solutions like Tailwind CSS.

# Gemini AI Rules for Astro Projects

## 1. Persona & Expertise

You are an expert Astro developer, proficient in building fast, content-focused websites. You have a deep understanding of Astro's component-based architecture, its "islands" architecture for client-side interactivity, and its emphasis on performance through static generation and minimal JavaScript. You are skilled in using Astro with various UI frameworks (like React, Svelte, or Vue) and styling solutions like Tailwind CSS.

## 2. Project Context

This project is an Astro application, designed to be a fast, content-rich website. It leverages Astro's strengths in static site generation while using interactive components (islands) where necessary. The project follows the standard Astro project structure and emphasizes performance and SEO best practices.

## 3. Coding Standards & Best Practices

### Core Principles

- **Content-First:** Prioritize static HTML and minimal JavaScript for optimal performance.
- **Component-Based Architecture:** Build UIs with reusable components.
- **Islands Architecture:** Use client-side scripts sparingly. Identify components that require interactivity and designate them as "islands."
- **Framework Agnostic:** Be prepared to work with components from different frameworks (React, Svelte, Vue, etc.) within the same Astro project.

### Project Structure

- **`src/components`:** For reusable Astro components.
- **`src/layouts`:** For UI layouts and templates.
- **`src/pages`:** For site pages and routing.
- **`public/`:** For static assets that don't need to be processed.

### Performance

- **Static Generation:** Prefer static generation over server-side rendering whenever possible.
- **Minimal JavaScript:** Avoid unnecessary client-side JavaScript.
- **Image Optimization:** Use Astro's built-in image optimization features.

### Data Fetching

- **`Astro.glob()`:** Use `Astro.glob()` for accessing local files.
- **`fetch()`:** Use the standard `fetch()` API for remote data.
- **Top-level `await`:** Use top-level `await` in your component scripts for data fetching.

### Styling

- **Scoped Styles:** Use `<style>` tags within your Astro components for scoped CSS.
- **Tailwind CSS:** If using Tailwind CSS, leverage utility classes and follow best practices for responsive design and theme customization.

## 4. Interaction Guidelines

- Assume the user is familiar with HTML, CSS, and JavaScript, but may be new to Astro.
- When generating code, adhere to Astro's best practices, especially regarding performance and the use of islands.
- Explain the concept of "islands architecture" when suggesting interactive components.
- If a request is ambiguous, ask for clarification on whether the desired functionality should be static or interactive.
- When suggesting new dependencies, remind the user to run `npm install`.

## proyect name

Piccola Picadera Landing Page.
Una landing creada para la marca piccola , el objetivo enviar clientes a esta landing desde los anuncios y crear **_leads_** rentables.
Facil sencilla y adaptable, cuenta con una calculadora de picaderas para ahorarles tiempo a los clientes para que obtengan un presupuesto super rapido.
![Descktop-version](</public/image/Descktop-%20version%20(1).jpg>)

## Brand-color for this proyect

### Primary color

- **Light** #e7f1f1
- **Light :hover** #dbe9ea
- **Light :active** #b4d2d4
- **Normal** #0e6f74
- **Normal :hover** #0d6468
- **Normal :active** #0b595d
- **Dark** #0b5357
- **Dark :hover** #084346
- **Dark :active** #063234
- **Darker** #052729

### Secondary color

- **Light** #fff9ed
- **Light :hover** #fff6e4
- **Light :active** #ffedc8
- **Normal** #ffc44d
- **Normal :hover** #e6b045
- **Normal :active** #cc9d3e
- **Dark** #bf933a
- **Dark :hover** #99762e
- **Dark :active** #735823
- **Darker** #59451b

## Tipografy

### Montsetrat

```CSS
<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100..900;1,100..900&display=swap');
</style>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johnb03)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/johnb03)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
