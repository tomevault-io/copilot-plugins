## ng-builders

> This repository contains a custom Angular CLI builder for building Chrome extensions. The project is written in TypeScript and is designed to be used within an Angular workspace.

# Project: ng-builders - Custom Angular Builder for Chrome Extensions

## Project Overview

This repository contains a custom Angular CLI builder for building Chrome extensions. The project is written in TypeScript and is designed to be used within an Angular workspace.

The primary builder, named `extension`, streamlines the development of Chrome extensions by:

1.  Executing the standard Angular application build (`@angular/build`) to compile the main application, which can be used as a popup or options page.
2.  Using `esbuild` to compile and bundle browser-specific scripts, such as `background.ts` and `content.ts`, into the final output directory.

This approach allows developers to leverage the power of the Angular framework for the UI components of their extension while managing the extension-specific scripts in a clean and integrated way.

## Key Files

-   `package.json`: Defines project metadata, dependencies (`@angular-devkit/architect`, `esbuild`, etc.), and build scripts.
-   `builders.json`: Registers the custom `extension` builder with the Angular CLI architect.
-   `src/extension-builder.ts`: The core implementation of the custom builder. It orchestrates the build process.
-   `src/schema.json`: Defines the configuration options available for the builder, such as paths to the `backgroundScript` and `contentScript`.
-   `tsconfig.json`: TypeScript compiler configuration for building the builder itself.

## Building and Publishing

The following scripts are available in `package.json` to manage the builder package:

-   **`npm run build`**: Compiles the TypeScript source code of the builder into JavaScript files in the `dist/` directory. This is the main command to build the project.
-   **`npm run clean`**: Deletes the `dist/` directory.
-   **`npm run publish:package`**: A convenience script that increments the package version, builds the project, and publishes it to the public npm registry.

### How the Builder is Used

This project produces an npm package. To use the builder, you would install this package in an Angular project and configure it in your `angular.json` file, similar to this:

```json
// In angular.json
"architect": {
  "build": {
    "builder": "@stempelv/ng-builders:extension",
    "options": {
      "backgroundScript": "src/background.ts",
      "contentScript": "src/content.ts",
      // ... other options
    }
  }
}
```

Then, running `ng build` would execute this custom builder.

## Development Conventions

-   **Language:** TypeScript
-   **Package Manager:** The presence of `pnpm-lock.yaml` suggests `pnpm` is the preferred package manager.
-   **Code Style:** The `tsconfig.json` is configured with `"strict": true`, enforcing strong type-checking.
-   **Bundling:** `esbuild` is used for its performance in bundling the background and content scripts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vladimirstempel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vladimirstempel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
