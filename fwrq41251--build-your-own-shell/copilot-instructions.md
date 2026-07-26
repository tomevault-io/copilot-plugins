## build-your-own-shell

> This project is a Java-based solution for the "Build Your Own Shell" challenge from CodeCrafters. The goal is to create a POSIX-compliant shell capable of interpreting commands, running external programs, and implementing built-in commands like `cd`, `pwd`, and `echo`.

# Gemini Code Assistant Context

## Project Overview

This project is a Java-based solution for the "Build Your Own Shell" challenge from CodeCrafters. The goal is to create a POSIX-compliant shell capable of interpreting commands, running external programs, and implementing built-in commands like `cd`, `pwd`, and `echo`.

The project is structured as a standard Maven project. The main application logic is located in `src/main/java/org.winry.Main.java`.

## Building and Running

The project includes a shell script, `your_program.sh`, which handles both compilation and execution.

### Building the Project

To build the project, the following Maven command is used:

```sh
mvn -q -B package -Ddir=/tmp/codecrafters-build-shell-java
```

This command compiles the Java source code and packages it into a self-contained JAR file named `codecrafters-shell.jar`, which is placed in the `/tmp/codecrafters-build-shell-java` directory.

### Running the Project

To run the application, the following command is used:

```sh
java --enable-preview -jar /tmp/codecrafters-build-shell-java/codecrafters-shell.jar
```

The `your_program.sh` script automates both the build and run steps. You can execute it directly:

```sh
./your_program.sh
```

## Development Conventions

*   **Language:** The project is written in Java.
*   **Build Tool:** Maven is used for dependency management and building the project.
*   **Java Version:** The `pom.xml` specifies the use of Java 25 and enables preview features.
*   **Project Structure:** The project follows the standard Maven directory layout.

---
> Source: [fwrq41251/build-your-own-shell](https://github.com/fwrq41251/build-your-own-shell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
