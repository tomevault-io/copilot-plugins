## bridge

> When using the `slog` logger, you should be saving it to a variable called `logger` and attach any relevant context

# Logging

When using the `slog` logger, you should be saving it to a variable called `logger` and attach any relevant context
using the `With` func.

# Rebuilding

When told to "rebuild" you should rebuild both the native binary to the root of the repo and the linux binary in
`dist/bridge-linux`

# Protobuf

Always use `make` to generate protobuf outputs (Go, TypeScript, JSON Schema). Never run `buf generate` directly.

---
> Source: [vercel/bridge](https://github.com/vercel/bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
