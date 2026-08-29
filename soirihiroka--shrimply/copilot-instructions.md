## shrimply

> Keep things simple and stupid:

Keep things simple and stupid:

1. Does this need to exist? -> no: skip it (YAGNI)
2. Stdlib does it? -> use it
3. Native platform feature? -> use it
4. Installed dependency? -> use it
5. One line? -> one line
6. Only then: the minimum that works

Avoid writing tests unless requested.

Run `make check` whenever a big change is done.

Never attempt to build inside the sandbox.

Avoid OOP, particularly inherentance unless absolutely necessary.

DO not use WebKit unless is is literally for rendering a webpage. Avoid using JS.

Avoid trivial functions and abstractions.

Always run make check outside of the sandbox to avoid blocking.

You are not allowed to use [allow(deprecated)]

Focus on your task.
You are not allowed to revert changes made by others unless explicitly requested.

Use fraction instead of f64 when dealing with time.

Never revert changes made by other agents.

A clear crash is always preferable to an unclean state

Put math logic into the math module. Use num_trait when fit.

You are not allowed to call cargo directly. Use Makefile.

CUDA Oxide supports many Rust features like enum and match. Use those when needed.

Avoid magic numbers.

Use builder pattern where it fits. Avoid crazy long function arguments.

Only bump project version when there is a need for it.

Never implement placeholder of any kind. It either works or it just fails entirely.

Avoid editing the shrimply file directly, and instead opt to use the shrimply MCP.
Inform the user when the MCP is missing certain features that are needed
instead of writing the file directly.

---
> Source: [soirihiroka/shrimply](https://github.com/soirihiroka/shrimply) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
