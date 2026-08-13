## caddy-language-server

> This project is a Caddy language server implemented in Golang.

This project is a Caddy language server implemented in Golang.

The source of truth for Caddyfile parsing is Caddy's source code and specifically Caddyfile parsing in https://github.com/caddyserver/caddy/tree/master/caddyconfig/caddyfile these should be imported directly so that the language server is always in sync with the Caddy's source code.

Caddyfile format is documented here https://caddyserver.com/docs/caddyfile. It needs to work correctly with directives and subdirectives described here https://caddyserver.com/docs/caddyfile/directives#syntax, request mathers described here https://caddyserver.com/docs/caddyfile/matchers#syntax and snippets described here https://caddyserver.com/docs/caddyfile/concepts#snippets

---
> Source: [teemuteemu/caddy-language-server](https://github.com/teemuteemu/caddy-language-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
