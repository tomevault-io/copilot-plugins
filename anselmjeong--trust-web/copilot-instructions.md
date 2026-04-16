## trust-web

> 1. **Use Pure Python with Reflex Components**: Write all frontend and backend code in Python using Reflex components (`rx.*`). Avoid JavaScript, HTML, or CSS unless wrapping custom React components. Use `rx.cond` for conditional rendering and `rx.foreach` for loops in UI, placing other logic in `State` classes. Example: Use `rx.hstack`, `rx.button`, `rx.heading` for layouts.

# Ensures code aligns with Reflex's pure-Python web app development principles and latest API
# Reference: https://reflex.dev/docs/

1. **Use Pure Python with Reflex Components**: Write all frontend and backend code in Python using Reflex components (`rx.*`). Avoid JavaScript, HTML, or CSS unless wrapping custom React components. Use `rx.cond` for conditional rendering and `rx.foreach` for loops in UI, placing other logic in `State` classes. Example: Use `rx.hstack`, `rx.button`, `rx.heading` for layouts.

2. **Follow Reflex App Structure**: Structure apps with a clear separation of concerns:
   - Import Reflex as `import reflex as rx`.
   - Define a `State` class (`rx.State`) for all dynamic variables (`vars`) and event handlers.
   - Create UI in functions (e.g., `index()`) using Reflex components that reference `State` vars and bind events (e.g., `on_click=State.event_handler`).
   - Initialize the app with `rx.App` and add pages via `app.add_page`.

3. **Leverage Latest Reflex API**: Always use the latest Reflex components and features from https://reflex.dev/docs/getting-started/introduction/. Prefer the 50+ built-in components (e.g., `rx.vstack`, `rx.form`, `rx.image`). For custom needs, wrap React components as per Reflex docs. Use `rx.el` for base HTML elements if needed.

4. **State Management**: Define all dynamic data in `State` or subclasses. Use event handlers to modify state, triggered by component events (e.g., `on_click`, `on_blur`). Ensure UI updates reactively by referencing `State` vars in components. Example: `rx.heading(State.count)` updates when `State.count` changes.

5. **Styling and Flexibility**: Style components using CSS props as keyword arguments (e.g., `color_scheme="grass"`, `class_name=["class1", "class2"]`). Support Chakra UI pseudo styles and theming (Reflex v0.4.0+). Use `rx.Theme` for app-wide theming, configurable via `rx.App` or `rx.ThemePanel`.

6. **Project Setup**: Initialize projects with `reflex init` and run with `reflex run`. Store static assets in the `assets/` folder and configure the app in `rxconfig.py`. Use virtual environments with Python 3.10+ for dependency management.

7. **Code Clarity and Best Practices**: Write clear, modular code. Nest components for complex layouts, use descriptive variable names, and document event handlers. Follow Reflex tutorials (e.g., dashboard, chatapp) for real-world patterns. Join Reflex Discord (#ask-ai) for community support.

8. **Deployment**: Use `reflex deploy` for single-command deployment. Ensure apps compile to standard frontend frameworks for flexibility, supporting small data apps to large multi-page sites.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AnselmJeong) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
