## sunudepot

> Audit complet - identifier incohérences, manques, doublons, problèmes d'affichage et réparer en profondeur (parcourir tous les répertoires et fichiers)


# Audit complet du projet

Quand l'utilisateur demande un **audit complet**, d’identifier les **incohérences**, les **manques d’information**, les **doublons** ou les **problèmes d’affichage/fonctionnalité**, tu dois :

1. **Lire et appliquer intégralement** le fichier **`PROMPT-AUDIT-COMPLET.md`** à la racine du projet (gestion-stock-btp).

2. **Explorer en profondeur** :
   - `app/Http/Controllers` (toutes les méthodes et variables passées aux vues)
   - `app/Models` (fillable, relations, accesseurs)
   - `app/Policies` (cohérence avec les rôles)
   - `routes/web.php` (noms, paramètres, middleware)
   - `resources/views` (toutes les vues : variables utilisées, route(), @csrf, old(), @error)
   - Fichiers de configuration et migrations si nécessaire

3. **Identifier** sans te limiter à la surface :
   - Incohérences (nommage, rôles, champs, statuts, messages)
   - Manques (variables non passées, champs manquants, autorisations, CSRF)
   - Doublons (code, vues, textes, validations)
   - Problèmes d’affichage et de fonctionnalité (liens, formulaires, rôles, prix)

4. **Réparer** chaque problème trouvé en modifiant les fichiers concernés, pas seulement en listant les recommandations.

5. **Produire** en fin d’audit le rapport structuré et la liste des modifications effectuées comme indiqué dans `PROMPT-AUDIT-COMPLET.md`.

Ne pas se contenter de quelques fichiers : parcourir les répertoires et ouvrir les fichiers pour vérifier contrôleurs ↔ vues ↔ modèles ↔ routes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lyissa1928-ai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lyissa1928-ai)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
