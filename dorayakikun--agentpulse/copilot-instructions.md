## agentpulse

> `scripts/codex-notify.sh` can be toggled with `CODEX_NOTIFY_DEBUG`.

## Codex notify debug toggle

`scripts/codex-notify.sh` can be toggled with `CODEX_NOTIFY_DEBUG`.

Enable for a single run:
```bash
CODEX_NOTIFY_DEBUG=1 codex "hello"
```

Enable persistently in `~/.codex/config.toml`:
```toml
notify = ["bash", "-lc", "CODEX_NOTIFY_DEBUG=1 /path/to/scripts/codex-notify.sh"]
```

Disable:
```bash
unset CODEX_NOTIFY_DEBUG
```

Log file: `/tmp/codex-notify-debug.log`

Note: Documentation comments must be written in English.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dorayakikun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
