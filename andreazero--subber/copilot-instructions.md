## 02-multiagent

> Single-lead agent workflow


# Multi-agent

Default: **lead unico**, un task, nessun subagent.

Subagent solo se due pezzi sono isolati e il lead ha già fissato i contratti (tipi, path, invoke). Un file, un owner. Nessuno inventa cartelle, dipendenze o task fuori ordine.

| Ruolo | Scope |
|---|---|
| Lead | Integrazione, task corrente, `AGENTS.md` |
| UI | `src/` React; invoke già definiti |
| Native | `src-tauri/`; FFmpeg, path, sidecar |
| ASR | `worker/` trascrizione + export sorgente |
| Translation | Traduzione su testo già trascritto |
| Subtitles | Formatter SRT + JSON; non cambia il significato |
| Review | Solo lettura, difetti concreti |

Mac: predisporre la build, non eseguirla da Windows.

---
> Source: [AndreaZero/subber](https://github.com/AndreaZero/subber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
