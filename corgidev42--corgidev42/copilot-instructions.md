## corgidev42

> Ce dépôt est la page d’accueil GitHub : `README.md` à la racine.

# README profil (`Corgidev42/Corgidev42`)

Ce dépôt est la page d’accueil GitHub : `README.md` à la racine.

## Stacks (badges)

- Quand un **nouveau langage ou outil** apparaît dans tes projets (`~/Documents/Workspace`, `~/Documents/Workspace-eve` en ignorant `eve-dev-env` et gros `node_modules`), **ajoute le badge shields.io** dans la section adaptée du README, sans inventer une techno que tu n’utilises pas.
- Retire les badges qui ne correspondent plus à ta pratique.

## Après une modification du README (ou du workflow profil)

1. **Commit / push** sur `main` → le workflow `.github/workflows/profile-automation.yml` se lance déjà au push.
2. Pour déclencher **sans commit** (cartes stats + serpent) :
   ```bash
   gh workflow run profile-automation.yml -R Corgidev42/Corgidev42
   ```
3. Pour rafraîchir le profil depuis **un autre dépôt** après un push : utiliser `repository_dispatch` (`event-type: profile-update`) comme documenté en tête de `profile-automation.yml`, avec un PAT en secret.

## Fichiers sensibles

Ne pas committer de secrets ; les cartes stats utilisent `GITHUB_TOKEN` dans Actions uniquement.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Corgidev42)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Corgidev42)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
