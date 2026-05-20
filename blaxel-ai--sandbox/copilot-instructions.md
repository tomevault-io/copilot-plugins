## fd-leak

> In this file sandbox-api/src/handler/process/process.go


In this file sandbox-api/src/handler/process/process.go
type ProcessInfo should never make a link to exec.Cmd cause this can cause FD leak
You should always use ProcessPid for link to CMD

---
> Source: [blaxel-ai/sandbox](https://github.com/blaxel-ai/sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
