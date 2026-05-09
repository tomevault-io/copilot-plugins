## py-unicode-golf

> - All logic must live inside functions. Scripts that are meant to be run directly must call a `main()` function at the bottom under `if __name__ == '__main__'`.

# Claude Code Instructions

## Code structure
- All logic must live inside functions. Scripts that are meant to be run directly must call a `main()` function at the bottom under `if __name__ == '__main__'`.
- Functions should be short and focused. If a function is getting long, split it. Exceptions: specific algorithms (Dijkstra loops, etc.) or cases where extracting sub-functions would genuinely make the code harder to follow.
- Don't over-engineer. If a helper would only ever be called once and makes the caller harder to read, just inline it.

## Naming
- No single-letter variable names. Exception: `i`, `j`, `k` in tight loops where the meaning is obvious from context.
- Variable and function names must be descriptive. A reader should understand what something holds/does without reading its implementation.
- Don't prefix things with `_` unless there is a real reason (e.g. a method that genuinely should not be called from outside a class). `for _i in range(...)` is nonsense — just call it `i` or give it a real name.
- Don't make functions private (`_name`) unless there is a strong reason. Default to public.

## Style
- Prefer clarity over brevity. Don't compress logic into one-liners just because Python allows it.
- Avoid redundant comments. Don't comment what the code already says clearly. Do comment *why* something non-obvious is done.
- Consistent formatting: spaces around operators, blank lines between logical sections of a function.

## Python specifics
- Avoid mutable default arguments (`def f(x=[])`).
- Prefer f-strings over `.format()` or `%`.
- Use `get()` on dicts when the key might be absent rather than `try/except KeyError`.
- Don't shadow built-ins (`list`, `type`, `input`, `id`, etc.).
- Keep imports at the top of the file. No inline imports except for optional heavy dependencies or circular import workarounds.

## Web / Frontend specifics (HTML, CSS, JS)
- Strict separation of concerns: HTML, CSS, and JS must live in separate files. Never embed CSS (`<style>`) or JavaScript (`<script>`) blocks inside HTML files unless explicitly instructed for a single-file prototype.
- No inline garbage: Never use inline styles (`style="..."`) or inline event handlers (`onclick="..."`). Bind all events in the JavaScript file using `addEventListener`.
- Modern JavaScript: Use `const` by default, and `let` only when reassignment is strictly necessary. Never use `var`. Prefer `async/await` over `.then()` chains.
- DOM Management: Use `document.getElementById()` for unique containers and `document.querySelector()` for others. Cache DOM lookups in variables if accessed multiple times; do not query the DOM inside tight loops.
- CSS best practices: Keep selectors flat. Prefer styling by class names rather than IDs or deep tag nesting. Use CSS variables (`:root`) for recurring colors, spacings, and fonts.
- Global Scope: Do not pollute the global `window` object with variables. Wrap execution logic in an IIFE, an ES module, or an initialization function called at the bottom of the script.
- Third-Party Libraries: When configuring heavy libraries (like charting, graphing, or 3D tools), extract massive configuration objects into separate constants or factory functions. Do not clutter execution logic with 50-line JSON config objects.

## What not to do
- Don't add features or refactor things that weren't asked for.
- Don't add error handling for cases that can't realistically happen.
- Don't add docstrings to functions whose name and signature already explain them.
- Don't over-abstract. Three similar lines of code is fine. A factory-factory is not.
- Don't use abbreviations in names unless they are universally understood (`url`, `db`, `id`, `n` for a math variable).

---
> Source: [incremen/py-unicode-golf](https://github.com/incremen/py-unicode-golf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
