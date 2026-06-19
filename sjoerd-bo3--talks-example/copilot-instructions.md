## talks-example

> Slidev Presentation Guidelines

Slidev Presentation Guidelines

Code Style and Structure

- Write concise, technical Markdown with accurate examples.
- Use Vue's composition API for reactive logic in slides.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`).
- Structure slides with clear sections, sub-sections, and visual hierarchy.
- Favor named exports for components and functions.
- Use lowercase with dashes for directory names (e.g., `themes/custom-theme`).

Markdown and Frontmatter Usage

- Use Markdown for all slide content; prefer frontmatter for slide-specific configurations.
- Utilize Slidev's extended Markdown syntax for enhanced features like code blocks, diagrams, and LaTeX.
- Avoid enums; use literal types or maps instead.
- Implement Vue components with TypeScript interfaces for props.

Syntax and Formatting

- Use the `---` separator to define slide boundaries.
- Write declarative Markdown with clear and readable structure.
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.

UI and Styling

- Use Slidev's built-in themes or create custom themes for consistent styling.
- Implement responsive design with a mobile-first approach.
- Ensure styling consistency across slides.
- Utilize Slidev's theming capabilities for consistent design across platforms.

State Management and Data Fetching

- Use Vue's reactivity system (`ref`, `computed`, `watch`) for state management within slides.
- Use Vue's `onMounted` lifecycle hook for data fetching in interactive demos.
- Minimize the use of `watchEffect`; favor computed properties and memoization when possible.

Error Handling and Validation

- Prioritize error handling and edge cases in interactive components.
- Handle errors and edge cases at the beginning of functions.
- Use early returns for error conditions to avoid deep nesting.
- Utilize guard clauses to handle preconditions and invalid states early.
- Implement proper error logging and user-friendly error messages.
- Use custom error types or factories for consistent error handling.

Performance Optimization

- Optimize for both web and mobile performance.
- Use Vue's `defineAsyncComponent` for lazy loading non-critical components.
- Implement lazy loading for non-critical components.
- Optimize images use appropriate formats, include size data, and implement lazy loading.

Monorepo Management

- Follow best practices using pnpm for monorepo setups.
- Ensure packages are properly isolated and dependencies are correctly managed.
- Use shared configurations and scripts where appropriate.
- Utilize the workspace structure as defined in the root `package.json`.

Follow Official Documentation

- Adhere to the official documentation for each technology used.
- For Slidev, focus on Markdown and Vue integration.
- Stay updated with the latest best practices and updates, especially for Vue and Slidev.

Output Expectations

- Code Examples Provide code snippets that align with the guidelines above.
- Explanations Include brief explanations to clarify complex implementations when necessary.
- Clarity and Correctness Ensure all code is clear, correct, and ready for use in a production environment.
- Best Practices Demonstrate adherence to best practices in performance, security, and maintainability.
- When creating icons always use the Phosphor Duotone Icons collection: https://icones.js.org/collection/ph-duotone-line/

The default supported layouts are as follows and you can find other layouts in the directory /layouts:

```
    ## Layouts

    This theme provides the following layouts:

    ### Title
    Usage:
    ```
    ---
    layout: intro
    ---

    # Presentation title

    Presentation subtitle

    <div class="absolute bottom-10">
    <span class="font-700">
        Author and Date
    </span>
    </div>

    ```
    ![intro](https://i.imgur.com/gnB4oa8.png)

    ---

    ### Title & Photo
    Usage:
    ```
    ---
    layout: intro-image
    image: '/image/path'
    ---

    <div class="absolute top-10">
    <span class="font-700">
        Author and Date
    </span>
    </div>

    <div class="absolute bottom-10">
    <h1>Presentation title</h1>
    <p>Presentation subtitle</p>
    </div>

    ```
    ![intro-image](https://i.imgur.com/976e8Hu.png)

    ---

    ### Title & Photo right
    Usage:
    ```
    ---
    layout: intro-image-right
    image: '/image/path'
    ---

    # Slide Title
    ## Slide Subtitle

    ```
    ![intro-image-right](https://i.imgur.com/dE1r2bg.png)

    ---

    ### Title, Bullets & Image
    Usage:
    ```
    ---
    layout: image-right
    image: '/image/path'
    ---

    # Slide Title
    ## Slide Subtitle

    * Slide bullet text

    ```
    ![image-right](https://i.imgur.com/llEB75J.png)

    ---

    ### Title & Bullets
    #### Layout used by default
    ![default](https://i.imgur.com/Glu7KWK.png)

    ---

    ### Bullets
    Usage:
    ```
    ---
    layout: bullets
    ---

    * Slide bullet text

    ```
    ![bullets](https://i.imgur.com/rvQJMMc.png)

    ---

    ### Section
    Usage:
    ```
    ---
    layout: section
    ---

    # Section Title

    ```
    ![section](https://i.imgur.com/vnL8XOB.png)

    ---

    ### Statement
    Usage:
    ```
    ---
    layout: statement
    ---

    # Statement

    ```
    ![satement](https://i.imgur.com/Em3e8g3.png)

    ---

    ### Big fact
    Usage:
    ```
    ---
    layout: fact
    ---

    # 100%
    Fact information

    ```
    ![fact](https://i.imgur.com/hPL7qOj.png)

    ---

    ### Quote
    Usage:
    ```
    ---
    layout: quote
    ---

    # "Notable quote"
    Attribution

    ```
    ![quote](https://i.imgur.com/DMpzz0g.png)

    ---

    ### Photo - 3
    Usage:
    ```
    ---
    layout: 3-images
    imageLeft: '/image/path'
    imageTopRight: '/image/path'
    imageBottomRight: '/image/path'
    ---
    ```
    ![3-images](https://i.imgur.com/Lun6FnS.png)

    ---

    ### Photo
    Usage:
    ```
    ---
    layout: image
    image: '/image/path'
    ---
    ```
    ![image](https://i.imgur.com/S9TQ2AZ.png)
```

---
> Source: [Sjoerd-Bo3/talks-example](https://github.com/Sjoerd-Bo3/talks-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
