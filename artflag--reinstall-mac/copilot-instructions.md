## reinstall-mac

> This repository contains a collection of scripts and configuration files designed to automate the setup of a developer's environment on a fresh macOS installation. It uses Homebrew to install command-line tools and GUI applications, and includes various configuration files for shell prompts, editors, and other tools.

# GEMINI.md: macOS Setup Environment

## Directory Overview

This repository contains a collection of scripts and configuration files designed to automate the setup of a developer's environment on a fresh macOS installation. It uses Homebrew to install command-line tools and GUI applications, and includes various configuration files for shell prompts, editors, and other tools.

The main goal is to provide a reproducible and version-controlled way to set up a personalized development workspace.

## Key Files

*   `install.sh`: This is the primary entry point script. It automates the installation of Homebrew, oh-my-zsh, and all the packages and applications listed in the `Brewfile`. It also orchestrates Git setup and the configuration of other tools.

*   `Brewfile`: This file lists all the software to be installed by Homebrew. It includes both command-line tools (e.g., `neovim`, `git`, `starship`) and macOS applications (e.g., `visual-studio-code`, `docker`, `figma`).

*   `setup-neovim.sh`: This script handles the specific setup for the Neovim editor. It installs the `vim-plug` plugin manager and a predefined set of plugins for a better editing experience, including themes and language support.

*   `starship/starship.toml`: This is the configuration file for the [Starship](https://starship.rs/) cross-shell prompt. It defines the appearance, layout, and information displayed in the terminal prompt, such as Git status, directory, and command duration.

*   `ghostty/config`: Contains the configuration for the [Ghostty](https://github.com/mitchellh/ghostty) terminal emulator.

*   `README.md`: This file provides manual instructions for the setup process. Critically, it contains a large list of shell aliases and environment variables that need to be manually added to the user's `~/.zshrc` file after the main installation script is run.

## Usage

The intended workflow for using this repository is as follows:

1.  **Clone the Repository:** Download the repository to the target macOS machine.

2.  **Run the Installer:** Execute the main installation script from the root of the repository:
    ```bash
    sh ./install.sh
    ```
    This will install all the software and apply most of the configurations automatically.

3.  **Manual Shell Configuration:** After the script finishes, copy the aliases and environment variable exports from the `README.md` file and add them to your shell's configuration file (e.g., `~/.zshrc` or `~/.bash_profile`).

This repository acts as a "dotfiles" manager, allowing for easy replication of a consistent development environment across multiple machines.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArtFlag)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ArtFlag)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
