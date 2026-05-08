## style

> Codebase styles and patterns


# Codebase styles and patterns

## General

- Write very concise and simple code. 
- Avoid very complex patterns.

## Frontend

- Store state in Zustand stores, following the patterns found in the stores/ directory. Only use useState when the state is 100% component-specific.
- Use the api.ts util for all API requests.
- Build using modular components that make sense as independent units. 
- Do not create files with multiple components, unless there is a main component and a very small supporting component that won't be used elsewhere.
- Pages defined in pages/ should have minimal code. They should at most contain some higher level state and reference child components. 

## Backend

- Follow existing API patterns from xxxx_api.py files when creating new endpoints.
- Make sure endpoints are secure.
- Ensure everything has static types.

---
> Source: [skaldlabs/skald](https://github.com/skaldlabs/skald) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
