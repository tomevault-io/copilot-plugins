## compose-glfw

> Compose GLFW is a JVM Compose host that runs Compose UI in a GLFW window instead

# Compose GLFW

Compose GLFW is a JVM Compose host that runs Compose UI in a GLFW window instead
of the default AWT/Swing desktop host.

## Project map

- `src/main/kotlin`: core Compose GLFW library.
- `compose-glfw-demo`: runnable demo application.

## Dev tool commands

- `mise run check`: run repository checks.
- `mise run fix`: apply automatic formatting fixes.
- `mise run build`: build all Gradle modules.
- `mise run docs`: generate Dokka API documentation.
- `mise run publish-local`: publish artifacts to the local Maven repository.
- `mise run demo`: run the demo application.
- `mise run demo:x11`: run the demo, forcing the X11 GLFW backend on Linux.

Run `mise tasks ls --all` for the complete task list.

---
> Source: [sargunv/compose-glfw](https://github.com/sargunv/compose-glfw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
