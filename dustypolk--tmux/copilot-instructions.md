## tmux

> To make tmux use the config file from this git repo location, create a symlink:

# tmux Configuration Setup

## Symlink Setup

To make tmux use the config file from this git repo location, create a symlink:

```bash
ln -s ~/.config/tmux/tmux.conf ~/.tmux.conf
```

Then reload tmux configuration:

```bash
tmux source-file ~/.tmux.conf
```

This keeps the config in the git repo while making it accessible to tmux at the standard location.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DustyPolk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DustyPolk)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
