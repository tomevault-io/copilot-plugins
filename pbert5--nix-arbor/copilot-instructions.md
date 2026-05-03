## nix-arbor

> This repository prefers declarative Nix workflows.

# Agent Working Agreement

This repository prefers declarative Nix workflows.

## Core Direction

- Default to declarative configuration over ad-hoc imperative scripts.
- Keep system behavior encoded in Nix whenever practical.
- Favor small, composable modules with clear ownership.

## Documentation Hygiene

- When adding or changing a feature, component, flake output, inventory surface,
  or operational workflow, update the relevant docs in the same change.
- Prefer updating existing docs when they already cover the area, and add a new
  focused doc when the feature needs usage notes, architecture notes, or
  operator guidance of its own.
- Do not leave implemented behavior documented only in plans, chat history, or
  code comments; promote current truth into `docs/`.
- When a plan document becomes partially implemented, document the implemented
  subset clearly and label remaining items as planned rather than present fact.

## Agent Skill Usage

- For Nix, NixOS, Home Manager, flakes, overlays, derivations, dev shells, or
  repo-layout work, consult `.github/skills/nixos/SKILL.md` first.
- For flake architecture, module-boundary decisions, cross-class composition, or
  the preferred design theory for the primary system flake and `experiments/`,
  also consult `.github/skills/dendritic/SKILL.md` as an appendix to the
  `nixos` skill.
- Load only the reference files needed for the current task from
  `.github/skills/nixos/references/` to keep context focused.
- Keep the discoverable skill under `.github/skills/nixos/` aligned with the
  source material under `skills/third-party/kettleofketchup-nixos/` when it is
  updated.
- Prefer the `dendritic` skill's aspect-oriented design theory when shaping the
  main flake or experiment-local flakes: thin entry points, feature-first
  modules, and composition across NixOS/Home Manager/Darwin where it helps.

## File Naming

- Never use `default.nix`.
- Prefer explicit filenames even when the filename repeats the parent directory
  name.
- Redundant names are acceptable if they make the tree easier to visually
  interpret.

## Auto-Import Hygiene

- This repo uses `import-tree` to auto-import active modules under `modules/`.
- If a file under `modules/` should not be auto-imported, place it under a path
  containing `/_`.

## Container Policy

- Podman is the backend of record.
- Prefer Nix-native container management (`virtualisation.oci-containers`)
  instead of compose-style orchestration.
- Avoid introducing Docker Compose or non-declarative container wrappers unless
  explicitly requested.
- We can skip `virtualisation.oci-containers` for programs that are one-shot
  runs.

## Experiment Isolation (Important)

- Unstable/prototype work must stay isolated under `experiments/`.
- For experiments, changes should live in the experiment flake and
  experiment-local files, not in the system flake by default.
- Do not wire experimental services/modules into `flake.nix`, host assemblies,
  or shared system roles until the experiment is declared stable.
- Promotion from `experiments/` to system-level modules should be an explicit,
  separate step.

## Flake Evaluation Hygiene

- Flakes ignore untracked files.
- Before evaluating, building, or rebuilding a flake change that depends on a
  newly created file, stage the required untracked files first.
- Stage them in the correct Git repo for the flake being evaluated.
- If nested Git repos exist, determine which repo owns the file instead of
  blindly staging from the current directory.
- Do not stage unrelated files just to make evaluation pass.

---
> Source: [pbert5/nix-arbor](https://github.com/pbert5/nix-arbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
