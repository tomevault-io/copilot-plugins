## myubuntuansible

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible playbook repository designed to automate Ubuntu system setup and maintenance. The primary playbook (`playbook.yml`) handles package management and installs essential development tools on localhost.

## Core Commands

### Running the Playbook
```bash
ansible-playbook --ask-become-pass playbook.yml
```

### Syntax Validation
```bash
ansible-playbook --syntax-check playbook.yml
```

### Dry Run (Check Mode)
```bash
ansible-playbook --check --ask-become-pass playbook.yml
```

## Architecture

The repository follows a simple Ansible structure:
- `playbook.yml` - Main playbook that targets localhost with become privileges
- Tasks are organized linearly within the playbook file
- Uses standard Ansible `apt` module for package management
- Designed for Ubuntu systems with sudo access

## Important Notes

- Playbook runs on `localhost` by default
- Requires `become: yes` (sudo privileges)
- User must have passwordless sudo or be prepared to enter password when prompted
- Targets Ubuntu systems specifically (uses apt package manager)
- Installs kernel headers matching the running kernel version using `ansible_kernel` variable

---
> Source: [cloverhsc/MyUbuntuAnsible](https://github.com/cloverhsc/MyUbuntuAnsible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
