## cs61b-learning

> Core Java sources live under `src/`, grouped by coursework modules (`hw00`, `lec01`–`lec06`) and a sandbox `Main.java`. Keep homework solutions with their cohort folder and add new lecture demos under a matching `lecXX` directory. Maven writes compiled class files to `target/`; IntelliJ may mirror outputs into `out/`. Treat both as generated—do not edit them directly. Place future automated tests under `src/test/java`, mirroring the package of the code under test.

# Repository Guidelines

## Project Structure & Module Organization
Core Java sources live under `src/`, grouped by coursework modules (`hw00`, `lec01`–`lec06`) and a sandbox `Main.java`. Keep homework solutions with their cohort folder and add new lecture demos under a matching `lecXX` directory. Maven writes compiled class files to `target/`; IntelliJ may mirror outputs into `out/`. Treat both as generated—do not edit them directly. Place future automated tests under `src/test/java`, mirroring the package of the code under test.

## Build, Test, and Development Commands
Run `mvn compile` to build against Java 17 and populate `target/classes`. Execute existing or new JUnit suites with `mvn test`; this leverages JUnit 5 and Google Truth configured in `pom.xml`. Use `mvn clean` before sharing work if you need to remove compiled artifacts. After compiling, launch quick experiments from the command line with `java -cp target/classes Main` (swap `Main` for any other class with a `public static void main`).

## Coding Style & Naming Conventions
Follow standard Java conventions: 4-space indentation, braces on the same line as declarations, and `UpperCamelCase` for classes. Methods and variables stay `lowerCamelCase`, and constants use `UPPER_SNAKE_CASE`. Keep files ASCII-only unless the logic demands otherwise. Align package statements with directory layout when you create reusable components; lecture demos that intentionally omit packages should remain isolated to avoid name clashes.

## Testing Guidelines
Write unit tests with JUnit 5 and Google Truth assertions. Name each file `*Test.java` and place it in `src/test/java/<package>/`. Target meaningful cases rather than exhaustive loops, and prefer descriptive method names such as `sortsBackwardsArrays()`. Run `mvn test` locally before committing, and re-run focused subsets via `mvn -Dtest=ClassNameTest test`.

## Commit & Pull Request Guidelines
Existing history favors concise, present-tense summaries (often in Chinese), e.g., `改个变量名`. Match that brevity while ensuring the change is understandable. When opening a pull request, include a short problem statement, bullet the solution, and note any follow-up work. Attach console output for build/test runs and link related assignments or issues so reviewers can trace context quickly.

---
> Source: [sonwwall/cs61b-learning](https://github.com/sonwwall/cs61b-learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
