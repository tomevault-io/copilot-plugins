## lifelog-backend-symfony

> PRE-COMMIT workflow — revue synchronisée avec Pixel-Perfect, SEO et Security avant tout commit. Blocage si Security n'a pas donné son feu vert.


# PRE-COMMIT

Dès que l'utilisateur tape le mot-clé **PRE-COMMIT**, lancer immédiatement la revue suivante.

## Déclenchement

- Mot-clé : **PRE-COMMIT** (insensible à la casse).
- Action : exécuter la revue complète sans demander de confirmation.

## Workflow

1. **Récupérer les fichiers modifiés**  
   Exécuter `git diff --staged --name-only` (et si besoin `git diff --staged`) pour lister le contenu des changements staged.

2. **Revue synchronisée avec les trois subagents**  
   Pour l’ensemble des fichiers modifiés, appliquer successivement (ou en synthèse) les critères de :
   - **pixel-perfect-auditor** : espacements 4/8px, typo, états (hover, disabled, Skeletons), mobile-first, micro-animations.
   - **seo-semantic-specialist** : sémantique HTML5, structure H1–H6, JSON-LD, A11y, Web Vitals.
   - **security-qa-sentinel** : validation des inputs, OWASP (SQL, XSS, CORS), pas de leak dans les erreurs, edge cases, offline/sync.

3. **Rapporter les problèmes**  
   Pour chaque sous-revue : liste des points à corriger avec propositions de correction concrètes (code ou commandes).

4. **Feu vert Security obligatoire**  
   **Ne pas laisser l’utilisateur committer tant que le Security Sentinel n’a pas donné son feu vert.**  
   - Si le security-qa-sentinel signale des vulnérabilités ou risques critiques : indiquer clairement « Commit bloqué — corriger les points Security ci-dessus ».
   - Une fois les correctifs Security appliqués et revus : annoncer « Feu vert Security — vous pouvez committer ».

## Résumé attendu

- Synthèse par subagent (Pixel-Perfect, SEO, Security).
- Liste priorisée des corrections (Critical → Warning → Suggestion).
- Statut final : **Commit autorisé** ou **Commit bloqué** avec rappel des actions Security restantes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hugoschellinger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
