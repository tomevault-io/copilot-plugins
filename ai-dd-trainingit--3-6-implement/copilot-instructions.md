## client-web

> Html and css code rules


# Web development rules

## HTML rules

- Use standard HTML semantic tags like : `article`, `section`, `nav`, `header`, `footer`, `aside`...
- Avoid `div` or `p` for layout and use proper semantic tags
- Add ARIA attributes for accessibility
- Add `id` `name` or `role` attributes to facilitate node selections

## CSS rules

- We use [PicoCSS](mdc:https:/picocss.com/docs) as a reset 
  - Do not create custom classes
  - PicoCSS will style semantic HTML
  - Customize variables at `/src/client/styles.css`

## Images

- Avoid svg and use emojis for icons

## Web APIs

- Use fetch utils at [fetch.utils.ts](mdc:src/client/shared/fetch.utils.ts) for HTTP API requests
- Avoid external libraries as much as possible

## Web Components

- Use derived HMLElements as Web Components (no lit nor any other lib or framework)
- Do not use shadow DOM
- Write the template in a private property using html helper string 
  ```ts
  const html = String.raw;
  this.#template = html`<div>...</div>`;
  ```
- Use the constructor to add the innerHTML and call repositories
- Use the `connectedCallback` lifecycle method to add event listeners
- Use the `disconnectedCallback` lifecycle method to remove event listeners
- Do not define the custom element at the component level, instead, export the class and define it at the parent level

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
