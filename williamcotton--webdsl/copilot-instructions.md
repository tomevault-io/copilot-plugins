## webdsl

> Test for memory leaks: make test-leaks

Tests: make test
Test for memory leaks: make test-leaks
Lint: make lint
Clean and run: make clean && make build/webdsl && build/webdsl app.webdsl

We use a per-request memory arena and the jansson API exposes custom allocators that we wire up to this allocator.

Use camelCase for variables and functions.

---
> Source: [williamcotton/webdsl](https://github.com/williamcotton/webdsl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
