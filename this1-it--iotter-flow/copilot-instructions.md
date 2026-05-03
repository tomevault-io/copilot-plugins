## iotter-flow

> `iotter-flow` is a multi-module Maven workspace. Core domain and integration code lives in modules such as `iotter-core`, `iotter-backend`, `iotter-cassandra`, `iotter-mqtt`, and `iotter-integration`. REST-facing code is split across `iotter-flow-rest`, `iotter-rest-endpoints`, `iotter-rest-billings`, and related endpoint/model modules.

# Repository Guidelines

## Project Structure & Module Organization
`iotter-flow` is a multi-module Maven workspace. Core domain and integration code lives in modules such as `iotter-core`, `iotter-backend`, `iotter-cassandra`, `iotter-mqtt`, and `iotter-integration`. REST-facing code is split across `iotter-flow-rest`, `iotter-rest-endpoints`, `iotter-rest-billings`, and related endpoint/model modules.

The Vaadin UI is split into `iotter-flow-ui-core`, `iotter-flow-ui-shim`, and `iotter-flow-ui`. Java sources follow the standard Maven layout under `src/main/java`; static assets live in `iotter-flow-ui/src/main/resources`, `src/main/webapp`, and `iotter-flow-ui/frontend`. Treat `iotter-flow-ui/frontend/generated` as generated output. Integration tests live in `iotter-flow-it/src/test/java`. Design notes and migration docs are kept in `docs/`.

## ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in .agents/_template/PLANS.md) from design to implementation. Write new plans to the .agents dir, Place any temporary research, clones, etc., in a .gitignored subdirectory of .agents. But for permanent features, open a new subdir under .agents, so for example for the orchestrator, we would use .agents/orchestrator/PLAN.md ... start by doing `cp -r .agents/_template .agents/orchestrator`. You need to remove the preamble of each plan, do not copy blindly. And if you need to take a different approach than TDD just for a particular plan, put that clearly in the plan.

## Build, Test, and Development Commands
Run commands from the repository root unless you need a single module.

- `mvn clean install -DskipTests`: build all modules quickly.
- `mvn clean install`: full multi-module build.
- `mvn -pl iotter-flow-ui -am spring-boot:run`: start the Vaadin UI and required modules locally.
- `mvn -pl iotter-flow-ui -Pproduction package`: build the production frontend bundle.
- `mvn -pl iotter-flow-it -Pintegration-tests verify`: run TestBench integration tests.

## Coding Style & Naming Conventions
Use Java 21 and existing Maven conventions. Source code uses 4-space indentation, `UpperCamelCase` for classes, `lowerCamelCase` for methods and fields, and package names under `it.thisone.iotter.*`. Keep module names and Spring/Vaadin config classes descriptive, for example `SecurityConfig` or `TomcatJndiConfig`.

Frontend resources in `iotter-flow-ui/frontend/src` and `frontend/styles` use lowercase, dash-separated filenames such as `gridstack-board.js`. No formatter or Checkstyle config is currently enforced in the build, so follow surrounding code closely.

## Testing Guidelines
Integration tests use Vaadin TestBench with JUnit 4 and the `maven-failsafe-plugin`. Name browser-driven tests `*IT.java`; keep supporting page objects and elements alongside them. No coverage gate is configured, so add focused tests for changed behavior and document any manual verification when UI work cannot be automated.

## Commit & Pull Request Guidelines
Recent history favors short, imperative or descriptive subjects like `Chart.js Fixes for Vaadin 14` and `refactor provisioning`. Keep commit titles brief, specific, and scoped to one change. For pull requests, include a concise summary, affected modules, setup or migration notes, linked issues if any, and screenshots for visible UI changes.

---
> Source: [this1-it/iotter-flow](https://github.com/this1-it/iotter-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
