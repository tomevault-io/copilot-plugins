## supabase-sveltekit

> Agile sprint workflow for the Mentore Manager project. Use when the user mentions tickets, sprint, projets, objectifs, backlog, what's next, en cours, à faire, prochainement, assigner, marquer comme fait, état du projet, or when about to implement a feature.


# Agile Workflow — Mentore Manager

Ce projet suit un workflow **Agile à sprints** piloté depuis Notion. Deux espaces complémentaires :

- **Suivi de projet** (sous Prototype Mentore Manager) : tickets, sprints, projets, objectifs. Source de vérité pour *quoi* travailler et dans quel ordre.
- **Prototype Mentore Manager** : prototype interactif de l'app. Source de vérité pour *comment* implémenter (UI, schéma DB, relations).

## Règles

1. **Pour toute question de suivi** (tickets, sprint, projets, objectifs, priorités, backlog) : utiliser le skill `suivi-de-projet`. Ne pas demander les noms de pages ou IDs Notion — ils sont dans le skill.

2. **Avant d'implémenter une fonctionnalité** :
   - Lire le ticket dans Notion (skill `suivi-de-projet`).
   - Chercher et lire la page prototype correspondante (skill `mentore-manager-notion`, section "Implémenter un ticket").
   - Ne pas coder sans avoir consulté le prototype.

3. **Après implémentation** : mettre à jour le statut du ticket dans Notion (skill `suivi-de-projet`, Workflow 4).

4. **Git** : chaque ticket = une branche (`feat/TCK-XX-slug`) créée depuis `develop` (skill `git-workflow`).

5. **Ne jamais modifier Notion sans confirmation explicite de l'utilisateur.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/curiousanthony) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
