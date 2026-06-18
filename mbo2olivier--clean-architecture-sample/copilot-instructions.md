## clean-architecture-sample

> Ce repo sert de socle méthodologique pour un futur repo mémoire CNSS orienté modélisation DDD + Clean Architecture.

# AGENTS

Ce repo sert de socle méthodologique pour un futur repo mémoire CNSS orienté modélisation DDD + Clean Architecture.

## À lire d'abord
- `docs/conventions/`
- `docs/agent/`
- `docs/templates/`
- `docs/cnss/`
- `docs/cnss-memory/target-memory-structure.md`

## Protocole de travail
1. Lire le modèle existant avant toute création ou mise à jour.
2. Comparer toute proposition avec la dernière version validée.
3. Signaler explicitement toute rupture ou conflit avec le modèle existant.
4. Séparer systématiquement :
- conventions observées
- hypothèses
- questions ouvertes

## Règles
- Ne casse jamais un modèle existant sans le signaler explicitement.
- N'invente pas une convention si elle n'est pas observée ou au moins justifiée comme hypothèse.
- Utilise les templates et le schéma agent fournis avant de produire un nouveau modèle.
- Si plusieurs styles coexistent, documente les variantes au lieu d'en supprimer une silencieusement.
- Applique les décisions de socle déjà actées, notamment `Aggregats`, `ValueObject`, la base `Cnss.Shared.Domain.Abstractions.ValueObject`, les identifiants métier via service de domaine et le commit explicite des repositories du projet métier cible.

---
> Source: [mbo2olivier/clean-architecture-sample](https://github.com/mbo2olivier/clean-architecture-sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
