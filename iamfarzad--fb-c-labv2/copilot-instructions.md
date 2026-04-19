## fb-c-labv2

> Enforce port management for local development servers

# Port Management Rule

- **WebSocket server** must always run on port **3001**.
- **Next.js** must always run on port **3000**.
- **Before starting either server after any code change:**
  - Check that **no process is running** on port 3000 or 3001.
  - If a process is found on either port, terminate it before proceeding.
- This ensures no port conflicts and reliable local development.

**Reference:**
- [server/live-server.ts](mdc:server/live-server.ts)
- [hooks/use-websocket-voice.ts](mdc:hooks/use-websocket-voice.ts)
- [middleware.ts](mdc:middleware.ts)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iamfarzad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
