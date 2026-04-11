## dotfiles

> - **Client (@User)**: Product Owner & Stakeholder.

# Project: Enterprise Dotfiles Automation

## 1. Core Philosophy & Governance
- **Client (@User)**: Product Owner & Stakeholder.
    - **Responsibility**: Define requirements, approve architectural changes, and perform User Acceptance Testing (UAT).
    - **Constraint**: **No Coding**. The Client relies entirely on the AI Team for implementation.
- **AI Team (@Gemini)**: Full-Stack Development Team.
    - **Responsibility**: Architecture design, implementation, testing, and documentation.
    - **Standard**: All outputs must be production-ready, strictly following industry best practices.

## 2. Engineering Standards (Best Practices)
- **Idempotency**: All setup scripts (e.g., `install.sh`, `setup.sh`) must be safe to run multiple times without side effects.
- **Modularity**: Configuration must be decoupled.
    - *Anti-Pattern*: A single 1000-line `.zshrc`.
    - *Pattern*: `zsh/aliases.zsh`, `zsh/exports.zsh`, loaded dynamically.
- **Portability**: Code must detect the OS (macOS/Linux) and adapt logic accordingly.
- **Documentation**: "Code tells you how, comments tell you why." All complex logic requires inline documentation in English.

## 3. Tech Stack & Constraints
- **Shell**: Zsh (via Oh My Zsh framework). Scripts must follow the [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html).
- **Editor**: Neovim (NvChad distribution). Customizations must reside in the `lua/custom` directory to ensure clean upgrades.
- **Multiplexer**: Tmux.
- **Package Management**: Homebrew (macOS/Linux).
- **Version Control**: Git. Commit messages must follow [Conventional Commits](https://www.conventionalcommits.org/).

## 4. Project Roadmap (Revised)
- [x] **Phase 0: Initialization**
    - [x] Define Agents & Protocols (`GEMINI.md`, `AGENTS.md`).
    - [x] Audit existing structure.
- [x] **Phase 1: Zsh Architecture**
    - [x] Modularize `.zshrc` (Split into aliases, exports, functions).
    - [x] Implement `gemini_load` for context injection.
    - [x] **Client Approval**: Phase 1 Deployed & Verified.
- [x] **Phase 2: Tmux (Oh My Tmux)**
    - [x] Install `gpakosz/.tmux` (Oh My Tmux) as a submodule.
    - [x] Configure `tmux.conf.local` for user preferences.
    - [x] Install Dracula theme for Tmux.
    - [x] Ensure `~/.tmux.conf` is symlinked to `.dotfiles`.
- [x] **Phase 3: Neovim & VS Code**
    - [x] Install Dracula theme for Neovim (NvChad).
    - [x] Symlink Neovim config to `.dotfiles`.
    - [x] Create a `vscode/` directory in dotfiles and symlink `settings.json` and `keybindings.json` to the VS Code user directory.
    - [x] Implement VS Code extension snapshot/restore scripts.
- [x] **Phase 4: JetBrains IDE Strategy**
    - [x] **Architectural Review**: Discuss how to unify settings (Fonts, Color Scheme, Vim Plugin) across IntelliJ, PyCharm, GoLand, while keeping environment-specifics separate.
    - [x] Implement the shared configuration strategy (.ideavimrc).
- [ ] **Phase 5: Consistency & Documentation**
    - [ ] Verify `setup.sh` handles all the above symlinks (Idempotency check).
    - [ ] Update `README.md` to reflect the new structure and installation steps.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oldfatcrab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/oldfatcrab)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
