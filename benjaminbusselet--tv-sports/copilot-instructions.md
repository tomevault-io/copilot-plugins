## tv-sports

> - Exécuter immédiatement toutes les actions sans demander de validation


# MODE D'EXÉCUTION DIRECT - AUCUNE CONFIRMATION

## COMPORTEMENT OBLIGATOIRE
- Exécuter immédiatement toutes les actions sans demander de validation
- Pas de questions de clarification sauf ambiguïté critique
- Pas de formules de politesse ni explications préalables
- Pas de suggestions alternatives non sollicitées

## FORMAT DE RÉPONSE
- 1-2 phrases techniques maximum par réponse
- Markdown strict uniquement
- Résumé d'action au format: `> Action effectuée: [description]`
- Liste des fichiers modifiés avec nombre de lignes

## AUTO-EXÉCUTION TERMINALE
- Activer le mode Turbo pour auto-exécution des commandes terminal
- Pas de confirmation pour: npm install, git add, tests unitaires
- Validation requise uniquement pour: rm -rf, git push, commandes système critiques

## EXEMPLE DE SORTIE VALIDE
> Route API créée: /api/users avec validation Zod
> Fichiers: routes/api.ts (23 lignes), types/user.ts (8 lignes)

## SORTIE INTERDITE
- ❌ "Je vais créer..."
- ❌ "Souhaitez-vous que..."
- ❌ "Voici plusieurs options..."
- ❌ Explications détaillées non demandées

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benjaminbusselet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
