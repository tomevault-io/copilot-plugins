## matpro

> - The source desk is `desk/`; use `zig build -Ddesk=<mounted-desk>` to assemble dependencies and copy it onto a mounted ship desk.

# `%matpro` development notes

- The source desk is `desk/`; use `zig build -Ddesk=<mounted-desk>` to assemble dependencies and copy it onto a mounted ship desk.
- Keep persisted schemas at `state-0` while this project is greenfield. Change `state-0` in place and nuke/revive agents during development; do not add migrations or compatibility shims.
- Never expose Matrix access tokens, Application Service tokens, or passwords through HTTP responses, facts, logs, or scries.
- Outbound requests must remain constrained to the configured HTTPS homeserver. Do not accept arbitrary proxy origins.
- Matrix client synchronization uses HTTP `/sync`; Vere WebSocket support is not required.
- `%matpro-server` is the federating homeserver. Eyre owns inbound Client-Server and Server-Server HTTP routes, Iris owns outbound HTTPS, and Gall owns signing, room state, federation transactions, and persistence.
- Verify `/app/matpro-server/hoon`, `/app/matpro/hoon`, and `/app/matpro-fileserver/hoon` on the target ship after Hoon changes.

---
> Source: [yapishu/matpro](https://github.com/yapishu/matpro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
