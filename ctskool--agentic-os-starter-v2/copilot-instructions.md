## agentic-os-starter-v2

> Help the member set up or improve their system without making them learn its folder layout. First determine their intent. Honor a choice they already made; do not ask it again.

# Agentic OS V2 starter — instructions for the coding agent

Help the member set up or improve their system without making them learn its folder layout. First determine their intent. Honor a choice they already made; do not ask it again.

- **Fresh install:** read `setup/PLAYBOOK.md` and follow its phases. A member can use an existing Obsidian vault for their first Agentic OS installation.
- **Upgrade an existing Agentic OS:** read `setup/UPGRADE.md` first. Run `node aos.mjs upgrade` to discover installations and vaults without changing them. This applies to V1, an older V2 starter, and a customized system.
- **Unclear setup request:** ask one plain question: "Are we setting up Agentic OS for the first time, or upgrading a version you already use?" These are the two onboarding choices. Choosing particular features happens inside the upgrade conversation.
- **Repair request** ("it's broken", "run the doctor"): repair the intended installation rather than reinstalling. If its identity is unclear, use upgrade discovery to identify it first. Follow Phase 6 of the playbook for that installation once the member has chosen the repair; explain that `--full` runs a real workflow.

An `obsidian-v2/.runtime/aos-setup.json` marker means setup ran before. It is evidence, not a reason to turn an explicit upgrade request into repair. If a fresh-install request resolves to an already installed system, explain the finding and resolve that choice before running setup again.

Hard rules:

- Never ask for a key or password in chat. Never read, print or copy `jev.json`, `bridge-auth.json`, `providers.json`, `.credentials.json` or `~/.claude/.env`. The user enters an OpenRouter key through the local page opened by `node aos.mjs jev-key`.
- Never overwrite existing notes or replace an existing vault's `.obsidian/` settings. No whole-vault backups, copying runtime/credential directories, or broad vault commits.
- Discovery is read-only: no fetch, pull, installation, health test, service stop or restart. A running monitor's non-secret location metadata helps identify an installation; it is not proof that the installation is healthy.
- Keep stock starter source maintained as a complete release. Do not patch `obsidian-v2/` or `jarvis-v2/` as an installation or repair workaround. The explicit exception is a member-approved custom integration plan in `setup/UPGRADE.md`, implemented in their owned project with scoped backups, tests and rollback. Such projects require manual future integration unless reconciled with the maintained starter.
- `update` is for a recognized, unmodified starter checkout. It refuses customized or unknown source before stopping services. Do not remove that guard or discard changes to force an update.
- Get the member's approval for the concrete upgrade plan before applying it, and obtain explicit permission for its service handoff before stopping or restarting a running installation. Use that installation's own scripts; leave unrelated speech services and recovery tasks alone.
- Services are local only (`127.0.0.1`, ports 3217–3221). Do not expose them. Only one installation can own those ports.

Commands: `node aos.mjs setup | upgrade | start | stop | status | doctor [--full] | jev-key | autostart on|off | update | dashboard`.

`upgrade` reports what is present; it does not apply changes. `update` changes a stock starter. Follow the appropriate guide instead of treating those commands as synonyms.

---
> Source: [ctskool/agentic-os-starter-v2](https://github.com/ctskool/agentic-os-starter-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
