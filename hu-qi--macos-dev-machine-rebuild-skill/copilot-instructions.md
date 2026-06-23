## macos-dev-machine-rebuild-skill

> Audit, rebuild, and verify a macOS development machine with an explicit plan, reviewed configuration changes, safe SSH/GPG handling, multi-runtime setup, and a redacted environment snapshot.


# macOS Dev Machine Rebuild Skill

Use this Skill when a user needs to initialize, migrate, rebuild, or standardize a macOS development environment. The desired outcome is not merely “tools installed”; it is a **reproducible, reviewable, verifiable, and recoverable** setup.

> Canonical source: this file. Keep generated copies for Claude Code, Cursor, Copilot, Gemini CLI, Windsurf, or other tools synchronized from this source rather than editing multiple variants independently.

## Attribution and scope

This is an independent, AI-agent-oriented implementation informed by Su Yang (soulteary), “从零重建 macOS 开发机：可复现的环境初始化流程” (2026-06-14). It is not a full-text reproduction. See [`NOTICE`](./NOTICE) for attribution and license scope.

## Non-negotiable safety rules

1. **Never request, print, commit, or upload** private SSH/GPG keys, recovery codes, tokens, passwords, or real credential exports.
2. **Do not use `curl | bash` by default.** Download installers to a temporary file, review the source and checksum/signature where available, then execute only with the user’s approval.
3. **Do not overwrite existing identity configuration.** Before modifying `~/.ssh`, `~/.gnupg`, `~/.gitconfig`, `.zshrc`, or `.zprofile`, inspect it, make a timestamped backup, and present the planned diff.
4. **Treat company-managed devices as constrained.** Stop and ask for the organization’s MDM, proxy, VPN, certificate, security, and package-source requirements before changing system settings.
5. **Prefer project-pinned runtimes.** Use the versions declared by the active project. Without a project constraint, choose a currently supported LTS/runtime release only after checking the official source.
6. **Run verification after every meaningful stage** and report both passed checks and remaining gaps.

## Required input

Collect only the information needed to build a plan. Do not ask for secrets.

```yaml
machine:
  chip: Apple Silicon | Intel
  macos_version: "for example: 15.x"
  shell: zsh
network:
  need_proxy: true | false
  proxy_url: "optional local proxy URL; never a credential-bearing URL"
identity:
  git_user_name: "display name"
  git_user_email: "commit email"
  existing_gpg_key: true | false
  existing_ssh_key: true | false
runtime_versions:
  go: "project-pinned version or omitted"
  node: "project-pinned version or omitted"
  python: "project-pinned version or omitted"
  python_env_name: "for example: dev"
preferences:
  package_manager: Homebrew
  node_manager: nvm | fnm | volta | project-tooling
  python_manager: Miniforge | uv | pyenv | project-tooling
```

## Execution protocol

### 1. Inventory before changing anything

Run the supplied audit first:

```bash
chmod +x scripts/*.sh
./scripts/bootstrap.sh
```

Record the following facts:

- macOS version and CPU architecture;
- whether Xcode Command Line Tools and Homebrew are present;
- current shell and shell startup files;
- availability of `git`, `gpg`, `ssh`, Go, Python, Node.js, npm, and Bun;
- existing Git identity and signing settings, without exposing private key material;
- device-management, proxy, VPN, and company-security restrictions.

### 2. Produce a short plan and checkpoints

Organize the work in this order. Do not install every tool just because it appears in the list.

1. System prerequisites: macOS updates and Xcode Command Line Tools.
2. Package manager: Homebrew, if permitted.
3. Shell baseline: Zsh, PATH, and only the plugins the user needs.
4. Developer identity: Git name/email, SSH authentication, and optional GPG signing.
5. Runtime managers: Go, Python, Node.js, Bun — only those required by current projects.
6. Project tooling and GUI applications.
7. Validation and snapshot export.

At each checkpoint, state what changed, what was verified, and what remains manual.

## Setup guidance

### System and Homebrew

Install or update Xcode Command Line Tools first:

```bash
xcode-select --install
xcode-select -p
git --version
clang --version
```

For Homebrew, obtain the installer from its official source, download it to a temporary file, inspect it, and only then run it. After installation, use the architecture-aware `brew shellenv` command provided by Homebrew instead of hard-coding a path.

Validate with:

```bash
brew --version
brew doctor
brew config
```

### Zsh

Keep shell startup files small and reviewable. Add only the initialization blocks required by tools actually installed. Before editing:

```bash
cp ~/.zshrc ~/.zshrc.backup.$(date +%Y%m%d-%H%M%S) 2>/dev/null || true
cp ~/.zprofile ~/.zprofile.backup.$(date +%Y%m%d-%H%M%S) 2>/dev/null || true
```

Use version-controlled dotfiles or a clearly marked block in startup files rather than appending duplicate lines on every run.

### SSH, GPG, and Git signing

Treat SSH authentication and GPG signing as separate systems:

```text
Git commit/tag signing  → GPG
Git/SSH remote access   → ssh-agent or gpg-agent SSH bridge
Git identity            → user.name + user.email
```

Install the public tools only after review:

```bash
brew install gnupg pinentry-mac
command -v gpg
command -v pinentry-mac
```

Before modifying `~/.gnupg/gpg-agent.conf`, preserve existing configuration:

```bash
mkdir -p ~/.gnupg
chmod 700 ~/.gnupg
if [ -f ~/.gnupg/gpg-agent.conf ]; then
  cp ~/.gnupg/gpg-agent.conf ~/.gnupg/gpg-agent.conf.backup.$(date +%Y%m%d-%H%M%S)
fi
```

Use the discovered pinentry path rather than assuming `/opt/homebrew` or `/usr/local`:

```bash
command -v pinentry-mac
```

Configure Git only after confirming the intended identity and signing key:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global commit.gpgsign true
git config --global tag.gpgsign true
git config --global gpg.format openpgp
git config --global user.signingkey <KEYID>
```

Verification must not expose secret keys:

```bash
git config --global --get user.name
git config --global --get user.email
git config --global --get user.signingkey
git config --global --get commit.gpgsign
gpg --list-secret-keys --keyid-format LONG
ssh-add -l
```

### Runtime management

Select one manager per runtime family unless the project requires otherwise.

| Runtime | Preferred decision rule | Verify |
| --- | --- | --- |
| Go | Follow `go.mod`, toolchain directives, or team policy. | `go version`, `go env GOPROXY` |
| Python | Prefer project-managed environments; isolate each project. | `python --version`, `which python` |
| Node.js | Follow `.nvmrc`, `package.json`, Volta, or project docs. | `node -v`, `npm -v` |
| Bun | Add only where a project uses Bun. | `bun --version` |

Avoid setting a global mirror, proxy, or runtime version without the user’s network and project requirements.

## Verification

Run the repository validator after setup:

```bash
./scripts/verify.sh
```

Minimum successful checks:

```bash
sw_vers
xcode-select -p
git --version
brew --version
gpg --version
git config --global --get user.signingkey
ssh-add -l
go version
python --version
node -v
npm -v
bun --version
```

A missing tool is not necessarily an error: report it as “not required / intentionally not installed” or “pending installation” based on the agreed plan.

## Export a snapshot

Use the provided export script only after reviewing its output location:

```bash
./scripts/export-snapshot.sh
```

The snapshot must contain environment metadata and reproducible package/runtime declarations only. It must not contain private keys, raw GPG exports, full Git configuration, credential helper configuration, or credential-bearing URLs.

## Troubleshooting

| Symptom | Likely cause | Safe next action |
| --- | --- | --- |
| `brew: command not found` | shell initialization not loaded | run the `brew shellenv` line supplied by Homebrew; reopen the terminal |
| `git` unavailable | Xcode CLI not installed or incomplete | re-check `xcode-select --install` and `xcode-select -p` |
| GPG signing does not prompt | pinentry path or agent state is wrong | verify `command -v pinentry-mac`; back up then inspect `gpg-agent.conf`; restart the agent |
| `No secret key` | signing key is absent or mismatched | confirm only the intended key ID and Git email; never export a private key into the repo |
| GitHub does not show “Verified” | public key or commit email mismatch | add the public key in the Git hosting account and verify the commit email |
| Node/Python/Go version differs | runtime manager not loaded or project pin ignored | inspect project version files and the active shell initialization |

## Completion criteria

A rebuild is complete only when the user can:

- open a new terminal and use the required runtimes without manual PATH repairs;
- clone, build, test, and commit a representative project;
- authenticate to required Git hosts without exposing credentials;
- create a signed commit or tag when signing is intentionally enabled;
- rerun `scripts/verify.sh` successfully;
- export and review a snapshot that contains no secrets.

---
> Source: [hu-qi/macos-dev-machine-rebuild-skill](https://github.com/hu-qi/macos-dev-machine-rebuild-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
