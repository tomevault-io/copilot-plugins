## callers

> *   **Semantic HTML5:**

# HTML Coding Guidelines

*   **Semantic HTML5:**
    *   **Principle:** Use HTML5 semantic elements to describe the structure and meaning of the content.
    *   **Application:** Employ elements like `<article>`, `<aside>`, `<nav>`, `<section>`, `<header>`, `<footer>`, `<main>`, `<figure>`, `<figcaption>` appropriately. Avoid using `<div>` or `<span>` when a more specific semantic tag is suitable.

*   **Accessibility (A11y):**
    *   **Principle:** Write HTML that is accessible to all users, including those with disabilities using assistive technologies.
    *   **Application:** Provide `alt` attributes for all `<img>` tags. Use ARIA (Accessible Rich Internet Applications) roles and attributes where necessary to enhance semantics for assistive technologies, especially for dynamic content and custom widgets. Ensure proper heading hierarchy (`<h1>` to `<h6>`). Ensure form elements have associated `<label>` tags.

*   **Validation:**
    *   **Principle:** Ensure HTML is well-formed and valid according to W3C standards.
    *   **Application:** Regularly validate HTML using tools like the W3C Markup Validation Service. Properly close tags, nest elements correctly, and use lowercase for tags and attributes for consistency.

*   **Conciseness and Readability:**
    *   **Principle:** Write clean, readable HTML. Avoid unnecessary elements or excessive nesting.
    *   **Application:** Use indentation to reflect element hierarchy. Comment complex sections or non-obvious structures. Strive for the simplest markup that achieves the desired structure and semantics.

*   **Separation of Concerns:**
    *   **Principle:** HTML should define the structure and content, CSS should handle presentation, and JavaScript should manage behavior.
    *   **Application:** Avoid inline styles (use external or embedded CSS). Minimize inline JavaScript event handlers (attach listeners via JavaScript files). Keep markup focused on what the content *is*, not how it *looks* or *acts*.

*   **Forms:**
    *   **Principle:** Create usable and accessible forms.
    *   **Application:** Use appropriate input types (`email`, `tel`, `number`, `date`, etc.). Group related fields with `<fieldset>` and `<legend>`. Provide clear instructions and validation feedback (though often enhanced with JS/CSS).

*   **Character Encoding & Language:**
    *   **Principle:** Specify character encoding and primary language for the document.
    *   **Application:** Include `<meta charset="UTF-8">` as the first child of the `<head>`. Set the `lang` attribute on the `<html>` tag (e.g., `<html lang="en">`). 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreasalomone) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
