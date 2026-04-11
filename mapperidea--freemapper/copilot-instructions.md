## freemapper

> This project is **FreeMind**, a free mind-mapping software written in Java using the Swing graphical toolkit. It allows users to create and edit mind maps, which are diagrams used to visually organize information.

# Gemini Code Understanding

## Project Overview

This project is **FreeMind**, a free mind-mapping software written in Java using the Swing graphical toolkit. It allows users to create and edit mind maps, which are diagrams used to visually organize information.

The application follows a Model-View-Controller (MVC) architecture:

*   **Model:** The `freemind.modes.MindMap` class represents the data structure of a mind map.
*   **View:** The `freemind.view.mindmapview.MapView` class is responsible for rendering the mind map on the screen. The main application window is the `freemind.main.FreeMind` class, which is a `JFrame`.
*   **Controller:** The `freemind.controller.Controller` class handles user input and updates the model and view accordingly.

The project uses **Apache Ant** as its build system. The main build file is `freemind/build.xml`.

## Building and Running

The project is built and run using Apache Ant. The following commands are available from the `freemind` directory:

*   **`ant dist`**: Compiles the source code and creates a distributable version of the application in the `dist` directory.
*   **`ant run`**: Compiles and runs the application from the source code.
*   **`ant test`**: Runs the unit tests.
*   **`ant clean`**: Deletes all generated files.

To run the application, you will need to have Apache Ant installed. Then, from the `freemind` directory, run the following command:

Before using the ant run command be sure to be on the right version of java to be able to compile it:

```
sdk use java 8.0.302-open
```

```bash
ant run
```

## Development Conventions

*   **Coding Style:** The code follows standard Java coding conventions.
*   **Internationalization:** The application is internationalized using resource bundles (`.properties` files) for different languages. The main resource file is `freemind/Resources_en.properties`.
*   **Plugins:** The application has a plugin architecture, allowing for the extension of its functionality. Plugins are located in the `freemind/plugins` directory.
*   **Dependencies:** The project's dependencies are located in the `freemind/lib` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mapperidea)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mapperidea)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
