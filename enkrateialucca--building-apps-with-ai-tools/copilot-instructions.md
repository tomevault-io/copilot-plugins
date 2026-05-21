## general

> Global Rules for Simple HTML/JS Apps (No Backend)


# General Guidelines for Simple Web Apps

## Code Quality
- Use ES6+ features (let/const, arrow functions, template literals)
- Keep functions short and single-purpose (max 20 lines)
- Use meaningful variable and function names
- Always handle errors gracefully (try/catch in JS)
- Keep HTML semantic and accessible
- Follow the single-file approach for simple apps

## File Organization
- **Single-file apps**: Generate one HTML file with embedded CSS and JavaScript
- **Multi-file apps**: Use `index.html`, `styles.css`, `script.js` structure
- **Complex apps**: Organize with subdirectories for `css/`, `js/`, `assets/`

## HTML Standards
- Use semantic HTML5 elements (header, main, footer, section, article)
- Add alt text to all images
- Use labels for all form inputs
- Include proper ARIA attributes for accessibility
- Use proper heading hierarchy (h1, h2, h3...)

## JavaScript Standards
- Use strict mode (`'use strict';`)
- Avoid global variables (use IIFE or modules)
- Prefer querySelector/querySelectorAll over older DOM APIs
- Use event delegation for dynamic elements
- Implement proper error handling with try/catch blocks
- Use modern async/await syntax for promises

## Example of Good Code:

<!--
  Example: Minimal Todo List App (Single HTML/JS/CSS File)
  - Semantic HTML5
  - Accessible form
  - ES6+ JavaScript
  - All code in one file
-->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Simple Todo List</title>
  <style>
    body {
      font-family: system-ui, sans-serif;
      background: #f9f9f9;
      color: #222;
      margin: 0;
      padding: 0;
    }
    main {
      max-width: 400px;
      margin: 2rem auto;
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.07);
      padding: 2rem;
    }
    h1 {
      font-size: 1.5rem;
      margin-bottom: 1rem;
    }
    form {
      display: flex;
      gap: 0.5rem;
      margin-bottom: 1rem;
    }
    input[type="text"] {
      flex: 1;
      padding: 0.5rem;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    button[type="submit"] {
      padding: 0.5rem 1rem;
      background: #0074d9;
      color: #fff;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    button[type="submit"]:hover {
      background: #005fa3;
    }
    ul#todo-list {
      list-style: none;
      padding: 0;
    }
    ul#todo-list li {
      padding: 0.5rem 0;
      border-bottom: 1px solid #eee;
    }
  </style>
</head>
<body>
  <main>
    <h1>Todo List</h1>
    <form id="todo-form" autocomplete="off">
      <label for="todo-input" style="display:none;">Add Item</label>
      <input id="todo-input" type="text" placeholder="Add a new task" required aria-label="Add a new task" />
      <button type="submit">Add</button>
    </form>
    <ul id="todo-list"></ul>
  </main>
  <script>
    document.addEventListener('DOMContentLoaded', () => {
      const form = document.querySelector('#todo-form');
      const list = document.querySelector('#todo-list');
      const input = document.querySelector('#todo-input');

      form.addEventListener('submit', (e) => {
        e.preventDefault();
        const value = input.value.trim();
        if (value) {
          const li = document.createElement('li');
          li.textContent = value;
          list.appendChild(li);
          input.value = '';
        }
      });
    });
  </script>
</body>
</html>

---
> Source: [EnkrateiaLucca/building-apps-with-ai-tools](https://github.com/EnkrateiaLucca/building-apps-with-ai-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
