## wagyu

> Wagyu is a user-mode WireGuard endpoint for `:gen_tcp` and `:gen_udp` sockets.

# Wagyu

Wagyu is a user-mode WireGuard endpoint for `:gen_tcp` and `:gen_udp` sockets.
It runs them on SmolNet, a userspace TCP/IP stack, so it needs no TUN device.
The design, implementation order and progress are tracked in
https://github.com/ausimian/wagyu/issues/2; each step's issue is the spec for
that work.

## Development

MAINTAINING.md covers the toolchain, checks, interop tests, benchmarks and
releasing. In short:

- Use Elixir 1.19.5 with Erlang/OTP 28.3 locally.
- Run `mix precommit` before committing. The wireguard-go interop tests in
  it need Go on the `PATH`; without Go they are skipped.
- Keep `@version` in `mix.exs` as the single source of truth.
- Add user-visible release notes to `RELEASE.md` using Keep a Changelog sections.

---
> Source: [ausimian/wagyu](https://github.com/ausimian/wagyu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
