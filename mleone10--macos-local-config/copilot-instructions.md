## macos-local-config

> Automated management and provisioning of MacOS systems using Ansible. This project configures system settings, installs software via Homebrew, sets up development environments, and manages dotfiles.

# MacOS Local Config

Automated management and provisioning of MacOS systems using Ansible. This project configures system settings, installs software via Homebrew, sets up development environments, and manages dotfiles.

## Project Overview

- **Core Technology:** Ansible
- **Target OS:** MacOS
- **Package Management:** Homebrew (Packages, Casks, Taps)
- **Shell Environment:** Zsh configuration via `.zshrc`
- **Security:** Ansible Vault for sensitive variables (e.g., Git email)

## Architecture & Roles

The project is structured into modular Ansible roles:

- **`homebrew`**: Bootstraps the Homebrew package manager and ensures it is updated and available in the shell path.
- **`common`**: Handles general system configuration:
    - Sets system hostname.
    - Configures macOS `defaults` (Dock behavior, screenshot location, etc.).
    - Installs general-purpose applications (Chrome, Discord, etc.).
- **`dev`**: Sets up the development environment:
    - Zsh aliases, prompt, and SSH agent configuration.
    - Dotfile management (`.vimrc`, `.colima/default/colima.yaml`).
    - Installation of development tools (Go, Docker, AWS CLI, Terraform, gemini-cli, claude-code).
    - Global Git configuration, including SSH-based commit signing.

## Key Files

- **`site.yml`**: The main entry point playbook. Executable script that runs all roles and performs final UI adjustments (Dock configuration).
- **`library/bootstrap`**: A shell script to install Xcode Command Line Tools and Ansible.
- **`library/addAppToDock`**: A custom script helper used by the playbook to manage Dock icons.
- **`roles/dev/vars/main.yml`**: Contains Vault-encrypted sensitive data.

## Usage

### Initial Setup
Before running the playbook for the first time, bootstrap the environment:
```bash
./library/bootstrap
```

### Executing the Playbook
Run the main configuration playbook. You will be prompted for the Ansible Vault password and your macOS user password (for `sudo` tasks).
```bash
./site.yml
```

## Development Conventions

- **Modularity**: New configurations should be added to the appropriate role or a new role should be created.
- **Secrets**: Always use Ansible Vault for sensitive information like emails, API keys, or personal identifiers.
- **Idempotency**: Ensure all tasks are idempotent so the playbook can be run multiple times without unintended side effects.
- **Shell Config**: Changes to shell environment should be made via `ansible.builtin.lineinfile` or `ansible.builtin.blockinfile` targeting `~/.zshrc`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mleone10)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mleone10)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
