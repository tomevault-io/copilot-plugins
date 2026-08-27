## ho

> Tools to make Python interfaces to HTTP services by dynamically generating

# ho - Http Objects

Tools to make Python interfaces to HTTP services by dynamically generating
functions from URL templates and OpenAPI specifications.

## What This Project Does

`ho` turns URL templates and OpenAPI specs into callable Python functions at
runtime. No code generation step — just `route_to_func(url_template)` and you
get a function with the correct signature, docstring, and HTTP handling.

See [README.md](README.md) for usage examples and detailed feature overview.

## Architecture

The core pipeline is: **URL template → OpenAPI 3.0.3 spec → `ju.oas.Route` → Python function**

### Module Map

| File | Purpose |
|------|---------|
| `ho/__init__.py` | Exports `route_to_func`, `routes_to_funcs` |
| `ho/base.py` | All core logic (~940 lines) |
| `ho/util.py` | `SimpleMappingNamespace`, `extract_placeholders` |

### Key Functions in `ho/base.py`

| Function | What it does |
|----------|-------------|
| `route_to_func(route, base_url, ...)` | Main entry point. Converts a route spec (URL string, OpenAPI dict, Route object, or tuple) into a callable Python function |
| `routes_to_funcs(spec, base_url, ...)` | Batch convert all routes in an OpenAPI spec to a dict of functions |
| `routes_to_namespace(spec, base_url, ...)` | Like `routes_to_funcs` but returns a namespace for `api.get_users()` style access |
| `url_template_to_openapi(url_template, ...)` | Parse a URL template into an OpenAPI 3.0.3 spec |
| `url_template_to_func(url_template, ...)` | Convenience: URL template directly to function |
| `default_route_maker(route_spec, ...)` | Normalize various input formats to `ju.oas.Route` |

### Dependencies

- **`ju`** — JSON/OpenAPI utilities (`ju.oas.Route`, `ju.oas.Routes`, `ju.json_schema.json_schema_to_signature`)
- **`requests`** — HTTP library

### How `route_to_func` Works Internally

1. `default_route_maker` normalizes input to a `ju.oas.Route`
2. `json_schema_to_signature` creates a Python `Signature` from the route's parameter schema
3. A closure is created that:
   - Binds call arguments to the signature
   - Separates path, query, and body parameters
   - Formats the URL with path params
   - Makes the HTTP request via `requests` (or a provided session)
   - Processes the response through `egress` (default: try JSON → text → binary)
4. The closure is decorated with the signature and given a proper name/docstring

### URL Template Syntax

- Path params: `https://api.example.com/users/{id}`
- Query params: `https://api.example.com/search?q={query}`
- Inline defaults: `{param:default_value}` (e.g., `{page:1}`, `{limit:10}`)

## Development

### Running Tests

```bash
pytest
```

### Package Info

- **Package name**: `ho` (PyPI: `pip install ho`)
- **Repository**: https://github.com/i2mint/ho
- **Build**: setuptools via `setup.cfg`
- **CI**: GitHub Actions (`.github/workflows/ci.yml`) — lint, test, publish to PyPI on main

### Code Style

- No type stubs or separate type files — types are inline
- Docstrings follow NumPy style (Parameters/Returns sections with dashes)
- Functions use `*` to force keyword-only args where appropriate
- Prefer flat module structure — everything important is in `base.py`

### Common Maintenance Tasks

- **Adding a new parameter to `route_to_func`**: Add it to both `route_to_func` and propagate to `default_route_maker` / `url_template_to_openapi` as needed
- **Changing response processing**: Modify `default_response_processor` or add new egress patterns
- **Supporting new input formats**: Extend `default_route_maker` with a new case

### What NOT Exported But Available

`routes_to_namespace`, `url_template_to_func`, `url_template_to_openapi`,
`extract_placeholders` are in `ho.base` / `ho.util` but not re-exported from
`ho/__init__.py`. Consider whether new functions should be exported there.

---
> Source: [i2mint/ho](https://github.com/i2mint/ho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
