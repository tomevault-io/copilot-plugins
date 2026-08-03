## vital-brief

> App iOS Expo (React Native + TypeScript + expo-router). Veille infirmière à **2 onglets**,

# Vital Brief — mémoire projet (Claude Code)

App iOS Expo (React Native + TypeScript + expo-router). Veille infirmière à **2 onglets**,
d'après la maquette Claude Design « Releve 2b » (crème, sauge, terracotta). Voir
**`README.md`** pour le détail complet.

## À savoir avant d'éditer

- App **entièrement native**. **2 onglets** dans une barre flottante en pilule
  (`src/components/FloatingTabBar.tsx`) :
  - `Journal` (`app/(tabs)/index.tsx`), `Mot du jour` (`app/(tabs)/mot-du-jour.tsx`,
    icône **fleur**).
- **Contenu quotidien** : les écrans lisent l'objet `Edition` (`src/content/types.ts`)
  fourni par `EditionProvider` (`src/content/EditionProvider.tsx`), qui télécharge
  `config.contentUrl` (JSON), met en cache, et retombe sur `sampleEdition` si
  indisponible. Gabarit + règles dans **`daily-content/`**. **Ne pas** remettre de contenu
  de veille en dur dans les écrans — passer par l'`Edition`.
- **Journal** = **une carte par rubrique** (6 rubriques, `Category` dans `types.ts`),
  chaque carte = **un article** cliquable ouvrant sa **source réelle** (l'app agrège,
  elle ne rédige pas). Le libellé + la couleur de rubrique sont dérivés de `category`
  côté UI (`CATEGORIES` dans `index.tsx`) — pas dans les données.
- **Mot du jour** : le choix se fait à la génération (voir `daily-content/GENERATION.md`) :
  terme utile à une infirmière, **jamais un terme présent dans `recent-words.json`**.
  La « fiche complète » se déplie via un état local (`useState`) dans l'écran.
- **Design tokens** : toujours passer par `src/theme.ts` (couleurs/tints/ombres) et
  `src/fonts.ts` (alias `fonts.displayExtra`, `fonts.sansBold`, …). Ne pas coder les
  couleurs/polices en dur dans les écrans.

## Vérifier

```bash
npm run typecheck                 # tsc --noEmit — doit passer
npx expo export --platform ios    # doit bundler sans erreur
```

## Conventions

- Palette « 2b crème » : papier `#F5EFE7`, encre `#2B2620`, accent sauge `#6E8B6A`,
  terracotta `#B05F3E` (rubrique Alertes). Cartes blanches à ombre douce.
- Polices : **Bricolage Grotesque** en affichage (« Vital », titres d'articles, le terme,
  numéros d'étapes), **Hanken Grotesk** partout ailleurs.
- Règle éditoriale : dans les brèves, **noms et chiffres précis** (texte, organisme,
  montant), jamais de descriptions vagues. Chaque article a une `url` + une `source`.
- Phase 2 : nouveaux onglets → ajouter un `Tabs.Screen` (`app/(tabs)/_layout.tsx`) + une
  entrée dans `TABS` (`FloatingTabBar.tsx`).

---
> Source: [pierreespy/vital-brief](https://github.com/pierreespy/vital-brief) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
