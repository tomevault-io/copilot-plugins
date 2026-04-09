## exemplecursor

> [YYYY-MM-DD HH:MM:SS] MODIFICATION

# Règles de Logging pour le Developpement

## Format des Entrées de Log

### 1. Modification de Fichiers Existants
```markdown
[YYYY-MM-DD HH:MM:SS] MODIFICATION
Fichier : {nom_du_fichier}
Type de modification : {type}
Description : {description_courte}
Détails :
- {détail_1}
- {détail_2}
```

### 2. Création de Nouveaux Fichiers
```markdown
[YYYY-MM-DD HH:MM:SS] CRÉATION
Fichier : {nom_du_fichier}
Type : {type_de_fichier}
Description : {description_courte}
Contenu initial :
- {description_du_contenu}
```

### 3. Suppression de Fichiers
```markdown
[YYYY-MM-DD HH:MM:SS] SUPPRESSION
Fichier : {nom_du_fichier}
Raison : {raison_de_la_suppression}
```

## Types de Modifications à Logger

1. **Modifications de Structure**
   - Changements dans l'architecture des dossiers
   - Ajout/suppression de fichiers
   - Déplacement de fichiers

2. **Modifications de Code**
   - Ajout de nouvelles fonctionnalités
   - Corrections de bugs
   - Optimisations
   - Refactoring

3. **Modifications de Configuration**
   - Changements dans package.json
   - Modifications des fichiers de configuration
   - Ajout/suppression de dépendances

4. **Modifications de Style**
   - Changements majeurs dans le CSS
   - Ajout de nouvelles variables
   - Modifications du thème

## Organisation du Fichier développement.log

1. **En-tête de Section**
   - Chaque nouvelle session de développement commence par une ligne de séparation
   - Date et heure de début de session

2. **Corps des Entrées**
   - Chronologique (plus récent en bas)
   - Une entrée par modification
   - Format selon le type d'opération

3. **Pied de Section**
   - Résumé des modifications majeures
   - État du projet après modifications

## Bonnes Pratiques

1. **Clarté**
   - Descriptions concises mais complètes
   - Utilisation de listes à puces pour les détails
   - Pas d'abréviations non standard

2. **Consistance**
   - Utiliser les formats définis ci-dessus
   - Maintenir la chronologie
   - Respecter l'indentation

3. **Pertinence**
   - Logger uniquement les modifications significatives
   - Grouper les modifications mineures connexes
   - Inclure les impacts potentiels des modifications

## Exemple d'Utilisation

```markdown
[2025-03-12 15:30:00] MODIFICATION
Fichier : styles.css
Type de modification : Style
Description : Mise à jour des variables de couleur
Détails :
- Ajout de nouvelles variables pour le mode sombre
- Modification des valeurs de couleur primaire
- Ajout de commentaires explicatifs
``` 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benoit-lavigne)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/benoit-lavigne)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
