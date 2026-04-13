## next-radix-shadcn-theme

> Ce fichier sert de point d'entrée pour les règles et directives Cursor utilisées dans ce projet.

# Règles Cursor pour Next-Radix-Shadcn-Theme

Ce fichier sert de point d'entrée pour les règles et directives Cursor utilisées dans ce projet.

## Organisation des règles

Pour une meilleure organisation et maintenance, les règles Cursor ont été séparées en plusieurs fichiers thématiques dans le dossier `.cursor/rules/`.

Veuillez consulter les fichiers suivants pour des directives détaillées :

- [Aperçu général et introduction](./.cursor/rules/README.md)
- [Style et structure du code](./.cursor/rules/code-style.md)
- [Conventions de nommage](./.cursor/rules/naming-conventions.md)
- [Utilisation de TypeScript](./.cursor/rules/typescript-guidelines.md)
- [UI et style](./.cursor/rules/ui-style-guidelines.md)
- [Optimisation des performances](./.cursor/rules/performance-optimization.md)
- [Requêtes de base de données et modèles](./.cursor/rules/database-guidelines.md)
- [Configuration Git-flow et conventions](./.cursor/rules/git-workflow.md)
- [Initialisation du projet](./.cursor/rules/project-setup.md)
- [Règles de traduction de documentation](./.cursor/rules/translate-guidelines.md)

## Règles principales

### Traduction de documentation

L'assistant doit proposer de traduire les fichiers de documentation du français vers l'anglais dans les cas suivants :

1. À la demande explicite de l'utilisateur
2. À la fin d'une session de travail, si des fichiers de documentation en français ont été modifiés

Pour les détails complets sur le processus de traduction, consultez [Règles de traduction de documentation](./.cursor/rules/translate-guidelines.md).

### Conventions de code

- Utiliser des tabulations pour l'indentation (pas d'espaces)
- Utiliser des guillemets simples pour les chaînes de caractères JavaScript/TypeScript
- Suivre les conventions de nommage camelCase pour les variables et fonctions
- Suivre les conventions de nommage PascalCase pour les composants React

### Messages de commit

- Les messages de commit doivent être en anglais
- Suivre le format conventionnel: `type(scope): message`
  - Exemple: `feat(auth): add login functionality`
  - Exemple: `fix(ui): correct button alignment on mobile`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pataco80) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
