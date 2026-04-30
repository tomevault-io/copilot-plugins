## jdr-world-builder

> Si tu ne trouve pas une information que tu pourrais obtenir en scannant, regardant les fichiers du projet, fait le sans demander.

# Guide de Code - JDR.coffee

Si tu ne trouve pas une information que tu pourrais obtenir en scannant, regardant les fichiers du projet, fait le sans demander.

## 2. Règles Générales de Code

### Programmation Fonctionnelle

- Utiliser uniquement des composants fonctionnels React
- Ne jamais utiliser de classes JavaScript/TypeScript
- Favoriser les hooks et fonctions utilitaires pures

### Organisation des Imports

- Grouper les imports par origine: externes, types, internes
- Privilégier la syntaxe d'import avec alias `~` (ex: `~/shared/ui/button`)

### Normes de Nommage

- Fichiers et dossiers: kebab-case
- Composants React: PascalCase
- Hooks: camelCase commençant par `use`
- Variables/Fonctions: camelCase
- Types/Interfaces: PascalCase

### Interdiction des Fichiers Index

- Ne jamais utiliser de fichiers `index.ts` ou `index.js` pour regrouper et réexporter des modules
- Importer directement depuis les fichiers sources

## 3. Composants React

### Structure

- Utiliser la syntaxe de fonction nommée pour les composants exportés
- Déclarer les types/interfaces des props avant le composant
- Séparer la logique de la présentation quand c'est pertinent

### Props

- Utiliser la décomposition des props dans les paramètres
- Définir les valeurs par défaut dans la décomposition
- Utiliser des types explicites pour toutes les props

### État Local

- Privilégier les hooks standards (`useState`, `useReducer`)
- Extraire la logique complexe dans des hooks personnalisés

## 4. Gestion de l'État

### React Query

- Utiliser React Query pour toutes les requêtes de données
- Définir des clés de requête cohérentes
- Structurer les hooks de requête par entité fonctionnelle

### Zustand (pour État Global Client)

- Organiser les stores par fonctionnalité
- Nommer les stores de manière cohérente (ex: `useWorldStore`)
- Limiter la taille et la portée des stores

## 5. Styling

### Tailwind CSS

- Utiliser les classes utilitaires Tailwind pour tous les styles
- Maintenir une organisation cohérente des classes
- Utiliser l'extension className pour les conditions complexes

### Composants UI

- Utiliser les composants shadcn/ui comme base
- Personnaliser via l'API de variants de shadcn ou des classes Tailwind
- Maintenir la cohérence visuelle dans toute l'application

## 6. Meilleures Pratiques

### Performance

- Utiliser `useMemo` et `useCallback` pour les calculs ou fonctions coûteuses
- Éviter les rendus inutiles avec React.memo quand c'est pertinent
- Optimiser les requêtes avec React Query

### Accessibilité

- Utiliser des balises sémantiques HTML
- Assurer des contrastes suffisants
- Inclure des attributs ARIA quand nécessaire

### Code propre

- Maintenir des fonctions de petite taille (< 50 lignes si possible)
- Extraire la logique réutilisable dans des hooks ou fonctions
- Supprimer systématiquement le code mort et les imports inutilisés

## 7. Qualité du Code et Linting

- Ne pas ajouter de commentaires paraphrasant ce que le code fait
- En cas de suppression de feature ne pas commenter le code le supprimer

### Linting automatique

- Exécuter régulièrement `npm run lint:fix` (au moins avant chaque commit)
- Cette commande corrige automatiquement les problèmes de style, formatage et imports inutilisés

### Types TypeScript

- Ne jamais utiliser le type `any` - c'est une violation des règles de linting
- Toujours spécifier des types précis
- Pour les cas dynamiques, préférer `unknown` à `any`, puis utiliser des assertions de type

### Variables inutilisées

- Ne jamais déclarer des variables qui ne sont pas utilisées dans le code
- Ne pas préfixer les variables avec `_` pour contourner le linter - supprimer complètement ces variables
- Pour les fonctions importées mais non utilisées, supprimer l'import

### Stratégies de correction

1. Pour les paramètres de fonctions callbacks: utiliser `_` comme nom de paramètre si le paramètre est obligatoire mais non utilisé
2. Pour les déstructurations: ne déstructurer que ce qui est nécessaire
3. Pour les fonctions/composants: supprimer complètement les paramètres inutilisés plutôt que de les préfixer

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shug0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
