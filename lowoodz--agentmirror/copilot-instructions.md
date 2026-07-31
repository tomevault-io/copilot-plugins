## pre-commit-hygiene

> Before git commit — scan for secrets, personal paths, and portability issues


# Pre-commit hygiene

Before **every** `git commit` (human or agent):

1. Run `./scripts/check-commit-hygiene.sh --staged` (or rely on `.githooks/pre-commit` after `./scripts/install-git-hooks.sh`).
2. Never stage: `config/test.env`, `config/smr.yaml`, `test_model_api_key.txt`, `dist/`, `test-data/`.
3. Never commit real API keys — use `api_key_env`, `config/test.env` (gitignored), or `config/test.env.example` placeholders only.
4. Never commit personal/machine paths:
   - macOS `/Users/<real-user>/…`
   - Windows `C:/Users/<real-user>/…` (use `SMR_WINDOWS_USER`, `SMR_GUEST_STAGING`, `windows-user` placeholders)
   - LAN IPs / `HostName` in repo (belong in `~/.ssh/config` + `config/test.env`)
5. Tests/scripts: no hardcoded OS usernames or machine paths; use `load_test_env.sh`, `vm-ssh.sh`, `windows-user` placeholders. Optional blocklist: copy `config/local-hygiene.env.example` → `config/local-hygiene.env` (gitignored).
6. Multi-platform: prefer env vars + documented defaults in `config/test.env.example`; avoid host-specific paths in Rust/Python/PS1 under `crates/` and `scripts/`.

If the check fails, fix the finding or unstage the file — do not bypass with `--no-verify` unless the user explicitly requests it.

---
> Source: [lowoodz/AgentMirror](https://github.com/lowoodz/AgentMirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
