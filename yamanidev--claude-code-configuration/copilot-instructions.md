## claude-code-configuration

> These instructions apply at all times, in every conversation, regardless of which skill (if any) is invoked.

# User-level instructions

These instructions apply at all times, in every conversation, regardless of which skill (if any) is invoked.

## Hard limits

Do not perform any of the actions below unless the user has explicitly instructed you to in the current conversation. If a task seems to require one of them, surface it as a question first — explain the consequence in one line — and wait for explicit approval.

- **Git state mutation**: `add`, `commit`, `push`, `stash`, `reset`, `revert`, `checkout` over dirty work, branch/tag creation or deletion, force pushes. Read-only inspection (`status`, `diff`, `log`, `show`, `blame`, `ls-files`) is fine.
- **Applying database migrations** to any database. Generating migration files (Django `makemigrations`, Alembic `revision --autogenerate`, etc.) is fine; running them (`migrate`, `upgrade`) is not.
- **Dependency changes**: installing, upgrading, or removing packages via `npm`/`yarn`/`pnpm`/`pip`/`poetry`/`cargo`/`go mod` and similar.
- **Environment files and secrets**: editing or printing `.env*`, credential files, kubeconfig contents, or anything containing tokens or keys.
- **Production**: running scripts against production, calling destructive endpoints, or touching production data. Even read-only commands warrant confirmation when the active credentials or context target production.
- **Weakening the safety net**: disabling, skipping, deleting, or weakening tests to make a change pass.
- **Infrastructure mutation**: `terraform apply`, `pulumi up`, `kubectl apply`/`delete`/`patch`/`scale`, `helm install`/`upgrade`/`uninstall`, mutating `aws`/`gcloud`/`az` commands, DNS or IAM changes. Read-only inspection (`terraform plan`, `kubectl get`/`describe`/`logs`, cloud `describe`/`list`/`get`) is fine.
- **Deployments and pipelines**: pushing to deploy branches, manually invoking CD, restarting services, draining nodes, rolling deployments.
- **System state mutation**: `apt`/`dnf`/`pacman` install/remove/upgrade, `systemctl start`/`stop`/`enable`/`disable`/`mask`, `mount`/`umount`, `mkfs`, `parted`/`fdisk`/`lvm` operations, kernel module load/unload, `sysctl -w`, user/group changes (`useradd`, `passwd`, `chown -R`), edits under `/etc` or `/boot`, GRUB changes.
- **Running as root or with `sudo`** unless the user has explicitly said to. Read-only inspection without sudo is fine and preferred.
- **Auth-relevant configuration**: firewall rules (`iptables`/`nft`/`ufw`), SSH config, PAM, sudoers. Misconfiguring these can lock the user out of their own machine — always confirm and explain the recovery path before suggesting changes.

File deletion and CI configuration edits are allowed when relevant — they're version-controlled and recoverable.

## Authoring conventions

- **Break multi-line comments, docstrings, and string literals at sentence or comma-free clause boundaries, never mid-sentence.** When prose in code spans more than one line, end each line at a natural stop — the end of a sentence, or a clause boundary that carries no comma — so no line severs a phrase mid-thought. A line ending on a comma is still mid-thought; rewrite rather than wrap. This holds in every language and for every kind of embedded prose: code comments, docstrings, and multi-line message, label, or help strings alike.
- **Name things with words, and let a name be as long as it needs to be.** Length is not a cost worth optimizing, and both failure modes come from pretending it is: inventing contractions to save typing (`kw`, `assoc`, `u`), and settling for a weaker name because the right one felt long. The only question is whether the name is what the thing is actually called in that language or domain — established names are right however short they are (`kwargs`, `ctx`, `id`, `i`). A name is too long only when its extra words repeat what the type, scope, or surrounding code already establishes; that is redundancy, not length. Names are read far more often than they are written, so shorthand trades a one-time saving for a permanent tax on every reader, human or model. Apply this to names you introduce, and match the surrounding code — do not rename existing shorthand as a drive-by.

---
> Source: [yamanidev/claude-code-configuration](https://github.com/yamanidev/claude-code-configuration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
