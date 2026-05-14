## zrm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZRM (Zenoh ROS-like Middleware) is a minimal, single-file communication middleware built on Zenoh, providing a clean and simple API inspired by ROS2 patterns. The entire implementation lives in `src/zrm/__init__.py`.

## Core Architecture

### Design Principles

1. **Single-file implementation**: Everything in `src/zrm/__init__.py`
3. **Protobuf-only**: All messages use protobuf serialization via `zenoh.ZBytes()`

## Important Constraints

1. **No dynamic attribute access**: Avoid `hasattr()`, `getattr()`, and `setattr()`. Use direct attribute access and explicit checks instead. Dynamic attribute access reduces code clarity and makes type checking difficult.
2. **Type validation**: All components validate message types at runtime with `isinstance()`
3. **Namespaced imports**: Use fully qualified names (one level) for clarity, except for `typing` module. Examples:
   - Use `pathlib.Path` not `Path`
   - Use `geometry_pb2.Pose` not `Pose`
   - Exception: `typing` imports can be direct (e.g., `from typing import Optional`)

### Service Message Pattern

All service definitions must use nested Request/Response messages:

```protobuf
message ServiceName {
  message Request {
    // request fields
  }

  message Response {
    // response fields
  }
}
```

The `ServiceServer` and `ServiceClient` classes validate that the provided service type has both nested messages at initialization.

---
> Source: [JafarAbdi/zrm](https://github.com/JafarAbdi/zrm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
