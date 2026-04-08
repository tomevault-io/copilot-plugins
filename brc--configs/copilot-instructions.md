## configs

> Migrate away from Spacemacs to a custom Emacs configuration that is based on the Spacemacs features that I use. Create a new Emacs config from scratch based on selected Spacemacs code, packages, and configurations. The new Emacs configuration will be faster and less buggy than Spacemacs itself because we will only be using a subset of the code.

# Goal

Migrate away from Spacemacs to a custom Emacs configuration that is based on the Spacemacs features that I use. Create a new Emacs config from scratch based on selected Spacemacs code, packages, and configurations. The new Emacs configuration will be faster and less buggy than Spacemacs itself because we will only be using a subset of the code.

# Project Layout

1. The "./home/spacemacs" directory is a Git submodule of the upstream Spacemacs code repository.

1. The "./home/.spacemacs" file is my Spacemacs configuration file.

1. The "./home/.emacs.d" directory contains the new (from-scratch) configuration.

1. Please ignore anything outside of the 3 paths mentioned above for this project.

# Decisions

1. The new configuration is modular, with "core" modules and "feature" modules.

1. `package.el` is used for package management.

1. All functions borrowed Spacemacs will retain their original names so that it's easier to copy-paste code, easier to know where a function originated from, and easier to diff against upstream code.

1. Our function namespacing strategy is:
  - Use "spacemacs/" for anything borrowed from Spacemacs (i.e., keep as-is);
  - Use "brc/" for custom functions; and
  - No prefix is needed for standard package functions.

  Exception: Only rename spacemacs/* functions to brc/* if you're significantly modifying the function's behavior.

1. Windows support should be removed; it is not needed.

1. An iterative approach is used to add one feature at a time to the new config.

1. After each feature is added, I test the configuration with "emacs --init-directory=/data/git/brc/configs/home/.emacs.d -nw". Do not test emacs with -Q because user-emacs-directory is ~/.emacs.d and it will conflict with the existing Spacemacs install.

1.  I am using terminal Emacs during this project (not GUI Emacs); we will test GUI frames later.

# Features from Spacemacs to be implemented

As you implement each feature below, it is expected that you will first look at how the feature is generally implemented in Spacemacs (e.g., it may be its own layer or it may be a bunch of core functions). Once you learn the entirety of how it is implemented, you and I will decide whether the complexity is manageable enough to implement the feature in the new configuration or whether alternative options should be considered.

The current list of features to implement from Spacemacs are:

[x] package management
[x] `which-key`
[x] Global keybindings unrelated to Evil
[x] Evil mode (large tasked split into phases, described below)
[x] Status line
[x] SPC leader key (How is this implemented in Spacemacs?)
[x] Default leader keys from spacemacs core bootstrap
[x] FIX: SPC s c: Wrong type argument: commandp, spacemacs/evil-search-clear-highlight
[x] Helm
[x] SPC SPC helm-M-x-fuzzy-matching
[x] Projectile
[x] Helm packages (such as swoop, wgrep, helm-projectile-ag, etc)
[x] SPC * (search project w/input)
[ ] SPC l (layouts-transient-state) (How does Spacemacs do it? What other options are there?)
[ ] SPC p l (helm-persp-switch-project)
[ ] Treemacs (plus my current Treemacs configuration)
[ ] Magit
[ ] LSP
[ ] Tree-sitter
[ ] Org-Mode
[ ] TODO: Implement the spacemacs functions in default-leader-keys.el?
[ ] A ton of toggles under SPC t (need to look at each one)
[ ] auto-completion
[ ] narrowing
[ ] SPC c y (spacemacs/copy-and-comment-lines)
[ ] gd (spacemacs/jump-to-definition)
[ ] gD (spacemacs/jump-to-definition-other-window)
[ ] SPC a u (vundo)
[ ] drag-stuff and J/K keybindings
[ ] K (spacemacs/evil-smart-doc-lookup)
[ ] Rifle search
[ ] SPC v (expand region)
[ ] Terraform
[ ] Themes
[ ] Spell check

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brc)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
