## common-rules

> - Must use Typescript

# Base Requirements

- Must use Typescript
- Must use Angular
- Must use Angular Material

# Repository Information

- Only runs on `https://aistudio.google.com/`
- It's an extension to extend the functionality of Google's AI Studio

# Features

- Adds an `Import Prompt` button to the navbar, right below `Create Prompt`
  - Finds the following:
    ```
    for (const el of document.querySelector(".nav-list").children) {
        if (el.innerText.toLowerCase().includes("create prompt")) { console.log(el); break; }
    }
    ```
    And then injects the Import Prompt button right after it.

---
> Source: [Dillonu/ai-studio-extended](https://github.com/Dillonu/ai-studio-extended) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
