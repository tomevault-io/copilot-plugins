## team-beletre-apps

> - **Vue individuelle** : Gestion des responsabilités par membre

# Architecture Frontend et Modules

## Applications principales

### Application Responsabilités ([apps/responsibilities/](mdc:apps/responsibilities/))
- **Vue individuelle** : Gestion des responsabilités par membre
- **Vue globale** : Vue d'ensemble par type d'activité
- **Composants** : [apps/responsibilities/components/](mdc:apps/responsibilities/components/)
- **Pages** : [apps/responsibilities/pages/](mdc:apps/responsibilities/pages/)
- **Documentation** : [apps/responsibilities/README.md](mdc:apps/responsibilities/README.md)

## Modules partagés ([modules/](mdc:modules/))

### Composants UI ([modules/components/](mdc:modules/components/))
- **Composants réutilisables** : Badges, headers, loaders, etc.
- **Import** : `import { ComponentName } from '/modules/components/component-name.js'`

### Store ([modules/store/](mdc:modules/store/))
- **Gestion d'état centralisée** : État global de l'application
- **Organisation** : Par domaine métier (ex: `responsibilities.individual`)

### Cache ([modules/cache/](mdc:modules/cache/))
- **Cache des données API** : Optimisation des performances
- **Organisation** : Par endpoint API

### Utilitaires ([modules/utils/](mdc:modules/utils/))
- **Fonctions helper** : Formatage, validation, calculs
- **Import** : `import { functionName } from '/modules/utils/utility-name.js'`

## Ressources communes ([shared/](mdc:shared/))

### Styles globaux
- **CSS principal** : [shared/css/](mdc:shared/css/) - Styles de base et composants
- **En-tête** : [shared/app-header.css](mdc:shared/app-header.css) - Styles de l'en-tête
- **Import** : `<link rel="stylesheet" href="../../shared/style.css">`

Utiliser les styles partagés autant que possible pour les nouveaux composants ou éléments d'interface des applications et des vues.
Ne jamais crée de style supplémentaire spontanément mais UNIQUEMENT sur demande.
Si besoin de styles supplémentaires, rapportez cette information dans le chat pour qu'elle soit traiter ultérieurement.

### Client API
- **Client API** : [shared/api-client.js](mdc:shared/api-client.js) - Fonctions d'appel API
- **Authentification** : Gestion centralisée de l'auth
- **Import** : `import { apiRequest, checkAuth } from '../../../shared/api-client.js'`

## Conventions de développement

### JavaScript
- **Modules ES6** : Utilisation d'imports/exports
- **Nommage** : `camelCase` pour les fonctions, `PascalCase` pour les classes
- **Organisation** : Un fichier par composant/fonctionnalité

### CSS
- **Mobile-first** : Design responsive avec breakpoints 768px et 480px
- **Classes** : `kebab-case` pour les noms de classes
- **Variables** : Utilisation de variables CSS pour la cohérence

### HTML
- **Structure sémantique** : Utilisation appropriée des balises HTML5
- **Accessibilité** : Respect des standards WCAG
- **Composants** : Structure modulaire et réutilisable

## Flux de données
```
User Action → Component → Store → Cache → API
                ↓         ↓       ↓
              UI Update  State   Data
```

## Points d'entrée
- **Application principale** : [index.html](mdc:index.html)
- **Documentation modules** : [modules/README.md](mdc:modules/README.md)
- **Documentation shared** : [shared/README.md](mdc:shared/README.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Etienne-TR)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Etienne-TR)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
