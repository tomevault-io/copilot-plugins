## passlet

> When standing up preview servers for the user to work from, don't run `npm run dev` or similar directly. Instead, use the `tportless` wrapper: bare `tportless` runs the project's dev script through the proxy, and `tportless <app-name> <command…>` works for any other server command. This exposes the preview server on the tailnet. Report the Tailscale URL from the tportless output back to the user, instead of the localhost URL.

# Agent instructions

## Preview servers

When standing up preview servers for the user to work from, don't run `npm run dev` or similar directly. Instead, use the `tportless` wrapper: bare `tportless` runs the project's dev script through the proxy, and `tportless <app-name> <command…>` works for any other server command. This exposes the preview server on the tailnet. Report the Tailscale URL from the tportless output back to the user, instead of the localhost URL.

---
> Source: [oscartrevio/passlet](https://github.com/oscartrevio/passlet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
