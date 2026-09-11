## swimmich

> <!-- mem0:brief v3 -->

# AGENTS.md

<!-- mem0:brief v3 -->
## Mémoire du projet

Une mémoire persistante (mem0) est branchée sur ce projet. Un rappel automatique
est injecté à chaque tour, mais il est calé sur la formulation de la demande : dès
que la conversation se déplace vers un sujet que ce rappel ne couvre pas, appelle
`mem0_search` **avant** de lire le code ou de proposer une solution. C'est moins
cher qu'une exploration de dépôt, et c'est la seule façon de retrouver un bug déjà
corrigé ou une décision déjà tranchée.

Appelle `mem0_search` en particulier avant de : débugger quelque chose qui
ressemble à du déjà-vu, trancher une question d'architecture, choisir une
convention de nommage ou de découpage, ou répondre à une question sur "comment on
fait ici".

**Mémorise** (`mem0_add`) : stack et choix techniques, décisions d'architecture
*avec leur raison*, conventions du dépôt qui ne sont écrites nulle part, bugs
résolus (symptôme + cause racine + correctif), exigences incontournables d'une
feature, préférences de travail exprimées par l'utilisateur.

**Ne mémorise pas** : l'état courant du code, ce qui est déjà écrit ici ou dans le
README, un raisonnement en cours, un résultat de test, du bavardage, un secret.

Un fait par appel, autoportant, rédigé tel quel — `mem0_add` stocke ton texte
sans le reformuler. Avant d'écrire, il cherche un souvenir proche : s'il en trouve
un, il le **complète** au lieu de créer un doublon, et te renvoie la version
fusionnée. Relis-la : si la fusion est mauvaise, réécris l'entrée avec
`mem0_update`.

Le dépôt fait toujours autorité contre un souvenir : s'il le contredit, le souvenir
est périmé — corrige-le (`mem0_update`) ou supprime-le (`mem0_forget`), ne
travaille pas dessus.

Règles complètes et exemples : `.omp/mem0-brief.md` — lis-le avant ton premier
`mem0_add` dans ce projet.
<!-- /mem0:brief -->

---
> Source: [millianlmx/swimmich](https://github.com/millianlmx/swimmich) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
