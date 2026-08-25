## ssh-hardening

> `src/modules/bbr.sh` is also published as the standalone `chnnic/BBR-tune` repository.

# Repository Instructions

## BBR synchronization

`src/modules/bbr.sh` is also published as the standalone `chnnic/BBR-tune` repository.

When `src/modules/bbr.sh` or a core helper used by that module changes:

1. Commit and validate `SSH-Hardening` first.
2. In a local `BBR-tune` checkout, run `scripts/sync-from-upstream.sh /path/to/SSH-Hardening`.
3. Update the standalone README when behavior or compatibility changed.
4. Run Bash syntax checks, ShellCheck, the sync check, and `tests/smoke.sh` in `BBR-tune`.
5. Commit and push both repositories in the same work item.

Do not copy the generated module block by hand. Do not place GitHub tokens in either repository or command history.

---
> Source: [chnnic/SSH-Hardening](https://github.com/chnnic/SSH-Hardening) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
