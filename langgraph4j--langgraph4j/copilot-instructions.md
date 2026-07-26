## langgraph4j

> This file provides local, task-focused guidance for coding agents working in this repository.

# LangGraph4j Agent Guide

This file provides local, task-focused guidance for coding agents working in this repository.

## Quick repo map

- `langgraph4j-core/` core runtime + public APIs.
- `langgraph4j-*/` optional modules (savers, integrations, observability, etc.).
- `langchain4j/` + `spring-ai/` adapters and compatibility layers.
- `samples/` runnable examples.
- `how-tos/` Jupyter notebooks (Java kernels via [rapaio-jupyter-kernel](https://github.com/padreati/rapaio-jupyter-kernel)).
- `studio/` UI and related tooling.
- `src/site/` documentation/website assets and site generation glue.

## Build and test

- Build all modules (skip tests): `./mvnw -q -DskipTests install`
- Run unit tests: `./mvnw -q test`
- Run a single module: `./mvnw -q -pl <module> -am test`

## Conventions and expectations

- Java 17+ only.
- Prefer additive changes to preserve public API stability.
- Follow module-local patterns (check the nearest module README).
- Keep logs structured and minimal in production code.
- Minimize third-party dependencies and avoid transitive bloat.
- Avoid non-ASCII changes unless the file already uses them.

## Before changing code

- Scan nearby modules for analogous patterns and tests.
- Use `rg` for search to keep edits precise.
- Avoid touching unrelated files in a dirty worktree.
- If modifying a public API, check if there is a BOM or adapter impact.

## After changing code

- Mention any tests you ran (or didn’t).
- If you add or change public APIs, update module README and relevant `how-tos/` docs.
- If you add a new module-level feature, ensure `how-tos/` covers it.

---
> Source: [langgraph4j/langgraph4j](https://github.com/langgraph4j/langgraph4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
