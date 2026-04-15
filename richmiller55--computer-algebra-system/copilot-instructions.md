## computer-algebra-system

> This document provides context for the Gemini code assistant to understand the project.

# Gemini Code Assistant Context

This document provides context for the Gemini code assistant to understand the project.

## Project Overview

This is a computer algebra system (CAS) written in Clojure. It provides functionalities for symbolic mathematics, including:

*   **Expression Simplification:** Simplifying mathematical expressions based on a set of rules (e.g., `x + 0 -> x`).
*   **Symbolic Differentiation:** Computing derivatives of functions.
*   **Symbolic and Numerical Integration:** Computing indefinite integrals and performing numerical integration using methods like the trapezoidal and Simpson's rule.

The project is in its early stages of development. The main entry point (`src/computer_algebra_system/core.clj`) is a placeholder, and the tests are not yet implemented. The core CAS logic is located in `src/computer_algebra_system/cas.clj` and `src/computer_algebra_system/integration.clj`.

The system represents mathematical expressions as Abstract Syntax Trees (ASTs) using Clojure's native data structures (lists for operations, symbols for variables, and numbers for constants).

## Building and Running

The project uses [Leiningen](https://leiningen.org/) as its build tool.

### Running Tests

To run the tests, use the following command:

```sh
lein test
```

### Running the Application

To run the application (which currently prints "Hello, World!"), use:

```sh
lein run
```

### Interactive Development (REPL)

For interactive development, you can start a REPL (Read-Eval-Print Loop):

```sh
lein repl
```

From the REPL, you can experiment with the CAS functions, for example:

```clojure
(require '[computer-algebra-system.cas :as cas])

(cas/simplify '(* (+ x 0) 1))
;; => x

(cas/differentiate '(* 2 x) 'x)
;; => (+ (* 2 1) (* x 0))

(cas/simplify (cas/differentiate '(* 2 x) 'x))
;; => 2
```

## Development Conventions

*   **Source Code:** The main source code is located in the `src/computer_algebra_system/` directory.
    *   `cas.clj`: Core CAS logic (simplification, differentiation).
    *   `integration.clj`: Symbolic and numerical integration.
    *   `examples_ast.clj`: Examples of the AST structure.
    *   `core.clj`: The main entry point of the application (currently a placeholder).
*   **Testing:** The project uses `clojure.test` for testing. Test files are located in the `test/computer_algebra_system/` directory.
*   **Dependencies:** Project dependencies are managed in `project.clj`. The only dependency is Clojure itself.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/richmiller55)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/richmiller55)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
