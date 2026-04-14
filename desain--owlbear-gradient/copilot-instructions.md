## owlbear-gradient

> This document provides an overview of the Owlbear Gradient codebase for Gemini invocations.

# Owlbear Gradient Codebase Overview

This document provides an overview of the Owlbear Gradient codebase for Gemini invocations.

## Project Structure

The project is a Typescript application using Vite for bundling. It's an extension for the Owlbear Rodeo VTT.

### Key Directories

-   `src`: Contains the main source code for the extension.
    -   `background`: The background script for the extension, which is a common pattern for extensions. The entry point is `src/background/background.html`.
    -   `popoverSettings`: Contains the code for the settings popover, which is built with React and Material UI. The entry point is `src/popoverSettings/popoverSettings.html`.
    -   `tool`: Contains the core logic for the gradient tool itself.
-   `assets`: Contains static assets like icons and CSS.
-   `public`: Contains public assets that are not processed by Vite.
-   `docs`: Contains documentation for the project.
-   `scripts`: Contains build scripts.
-   `test`: Contains test files.
-   `worker`: Contains a Cloudflare worker.

### Key Files

-   `vite.config.ts`: The configuration file for Vite. It defines the entry points for the build process.
-   `package.json`: Defines the project's dependencies and scripts.
-   `wrangler.jsonc`: The configuration file for the Cloudflare worker.
-   `tsconfig.json`: The base Typescript configuration.
-   `eslint.config.js`: The ESLint configuration.

## Development

The main development scripts are:

-   `pnpm dev`: Starts the development server.
-   `pnpm build`: Builds the extension for production.
-   `pnpm test`: Runs the tests using Vitest.
-   `pnpm lint`: Lints the code using ESLint.

## How it Works

The extension has two main parts:

1.  A background script (`src/background`) that communicates with Owlbear Rodeo.
2.  A settings popover (`src/popoverSettings`) built with React, and Material UI for state management. This is where users configure the gradient and patterns.

The core gradient logic is in the `src/tool` directory. The `install.ts` file likely handles the initial setup of the extension within Owlbear Rodeo.

## How Patterns Work

Patterns are a feature that allows users to apply repeating designs on top of the base gradient. The logic is distributed across three main files:

1.  `src/tool/GradientTarget.ts`: Defines the `Pattern` type as a union of string literals. This is the canonical list of available patterns.
2.  `src/popoverSettings/PatternSettings.tsx`: Renders the list of available patterns in the settings popover. It reads the pattern names from a `PATTERNS` constant and displays a `PatternPreview` for each.
3.  `src/popoverSettings/PatternPreview.tsx`: Contains a React component that renders an SVG preview for each pattern based on its name.
4.  `src/tool/GradientEffect.ts`: This is where the rendering logic for Owlbear Rodeo is constructed. It contains a `patternFunction` that maps a pattern name to a specific GLSL/SkSL shader function. The file includes an `sksl` string with the shader code for each pattern, which is executed by Owlbear Rodeo's graphics engine.

### Adding a New Pattern

To add a new pattern, you need to modify the files mentioned above. Here's a step-by-step guide:

1.  **Define the Pattern Name**: Open `src/tool/GradientTarget.ts` and add your new pattern's name to the `Pattern` type alias. For example, if you're adding a "DIAMOND" pattern:

    ```typescript
    export type Pattern =
        | "STRIPE"
        // ... other patterns...
        | "DIAMOND";
    ```

2.  **Add to the UI List**: Open `src/popoverSettings/PatternSettings.tsx` and add the new pattern name to the `PATTERNS` array.

    ```typescript
    const PATTERNS: Pattern[] = [
        "STRIPE",
        // ... other patterns...
        "DIAMOND",
    ];
    ```

3.  **Create a UI Preview**: Open `src/popoverSettings/PatternPreview.tsx` and add a new `case` to the `switch` statement. This should return an SVG that visually represents your pattern.

    ```tsx
    // ... inside the switch (pattern) block
    case "DIAMOND":
        return (
            <>
                {/* Your SVG elements here */}
            </>
        );
    ```

4.  **Implement the Shader**: This is the most critical part. Open `src/tool/GradientEffect.ts`:

    -   First, add a new `case` to `patternFunction` to map your pattern name to a new shader function name.

        ```typescript
        function patternFunction(pattern?: Pattern) {
            switch (pattern) {
                // ... other cases
                case "DIAMOND":
                    return "diamond"; // The name of your shader function
            }
        }
        ```

    -   Second, implement the `diamond` function in the large `sksl` template string. This function will receive `vec2 st` (texture coordinates) and should return a `vec4` color.

        ```glsl
            // ... inside the sksl string
            vec4 diamond(vec2 st) {
                // Your GLSL code to generate the pattern
                // Example: return mix(color(0), color(1), step(0.5, abs(st.s - 0.5) + abs(st.t - 0.5)));
            }
        ```

5.  **Verify your changes**. Run `pnpm build`, `pnpm lint`, and `pnpm test` to verify your changes. If any issues are reported, fix them and rerun.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/desain)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/desain)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
