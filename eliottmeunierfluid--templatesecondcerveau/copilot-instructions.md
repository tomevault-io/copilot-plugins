## templatesecondcerveau

> Ce dossier est un **WORKSPACE** : un conteneur qui peut héberger plusieurs projets côte à côte. Le vault principal (le « second cerveau ») est un de ces projets, dans le sous-dossier `Second Cerveau/`. D'autres projets (sites web, apps, repos client, etc.) peuvent être ajoutés à côté — chacun avec son propre `AGENTS.md` spécifique.

# WORKSPACE — Instructions globales

## Architecture

Ce dossier est un **WORKSPACE** : un conteneur qui peut héberger plusieurs projets côte à côte. Le vault principal (le « second cerveau ») est un de ces projets, dans le sous-dossier `Second Cerveau/`. D'autres projets (sites web, apps, repos client, etc.) peuvent être ajoutés à côté — chacun avec son propre `AGENTS.md` spécifique.

```
WORKSPACE/                     ← tu lances l'IA ici
├── AGENTS.md                  ← ce fichier — règles globales
├── CLAUDE.md                  ← @AGENTS.md
├── .claude/
│   └── skills/                ← skills transverses (lus depuis partout)
├── Second Cerveau/            ← le vault Obsidian (ouvert dans Obsidian)
│   ├── AGENTS.md              ← règles spécifiques au vault (IPCRA, Life Phases…)
│   └── [0 INBOX, 1 PROJETS, 2 CASQUETTES, …]
└── [autre-projet]/            ← autres repos/projets (optionnel)
    └── AGENTS.md
```

**Principe de cascade :** quand l'IA ouvre ce WORKSPACE, elle lit ce fichier + l'AGENTS.md de chaque sous-projet pertinent pour la tâche en cours. Pas besoin de répéter le contexte global dans chaque sous-projet.

---

## Identité

**Nom de l'IA :** [à définir lors de `/initialisation` — l'IA se présente et se réfère à elle-même sous ce nom]

**Prénom de l'utilisateur :** [à définir lors de `/initialisation`]

---

## À lire impérativement au début de chaque session

1. Ce fichier (règles globales)
2. `@Second Cerveau/AGENTS.md` — règles du vault principal (IPCRA, Life Phases, flux de ruissellement)
3. `@Second Cerveau/2 CASQUETTES/Sur ma vie/Moi.md` — qui est l'utilisateur, son style IA, ses valeurs
4. Selon la tâche : l'intention de la phase active, la dernière daily note, les notes de contexte des projets/casquettes concernés, et l'`AGENTS.md` du sous-projet si on travaille ailleurs que dans le vault

@Second Cerveau/AGENTS.md

---

## Style de communication

- Direct et concis, pas de blabla
- Respecter le style défini dans `Second Cerveau/2 CASQUETTES/Sur ma vie/Moi.md` (tutoiement, ton, niveau de pushback)
- TOUJOURS mettre les accents français (é, è, ê, à, â, ù, û, ô, î, ç)
- Utiliser des `[[wikilinks]]` généreusement dans les notes du vault
- Poser des questions si quelque chose n'est pas clair
- Reformuler pour vérifier la compréhension

---
> Source: [EliottMeunierFluid/templatesecondcerveau](https://github.com/EliottMeunierFluid/templatesecondcerveau) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
