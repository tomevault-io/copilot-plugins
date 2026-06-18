## learning-os

> Système d'apprentissage avec 5 modules — learning path planner (plan 1-3 mois), concept explainer (explication Feynman), study session designer (session 30-90 min avec livrable), knowledge synthesizer (synthèse de sources multiples), math rebuild (rattrapage maths avec diagnostic). Utilise cette skill dès que l'utilisateur dit "j'apprends X", "je veux maîtriser Y", "explique-moi Z", "je comprends pas X", "c'est quoi X vraiment", "je veux rattraper les maths", "par où commencer", "synthétise ces sources", "j'ai 1h pour étudier", ou demande d'expliquer un concept, construire un plan d'apprentissage, créer des fiches de synthèse. Se déclenche aussi quand l'utilisateur partage plusieurs articles/livres sur un sujet. Inclut toujours un mécanisme de vérification (test, livrable, question de contrôle) — pas de consommation passive. Produit des artefacts exportables pour continuité entre sessions.


# learning-os

Un système d'apprentissage pour transformer la curiosité en maîtrise, avec 5 modules qui insistent sur la **vérification** plutôt que la **consommation**.

## Philosophie

La plupart des gens confondent **exposition** et **maîtrise**. Ils lisent un livre, regardent une vidéo, et croient avoir appris. Ils n'ont rien appris — ils ont consommé.

`learning-os` applique 4 principes durs :

1. **Pas de plan sans test de sortie** — chaque plan inclut des checkpoints où tu prouves ce que tu sais
2. **Pas d'explication sans question de contrôle** — chaque concept expliqué se termine par une question-test
3. **Pas de session sans livrable observable** — exercices résolus, concept expliqué par écrit, code qui tourne
4. **Pas d'analogie sans sa limite** — les analogies sont utiles mais dangereuses ; la skill nomme toujours où elles cassent

## Les 5 modules

| Module | Rôle | Quand l'appeler |
|---|---|---|
| **Learning Path Planner** | Construire un plan 1-3 mois pour un domaine | Tu démarres un sujet (ML, droit, finance, autre) |
| **Concept Explainer** | Expliquer UN concept avec méthode Feynman + analogies + test | Tu bloques sur un concept précis |
| **Study Session Designer** | Structurer une session 30-90 min avec objectif | Tu as du temps pour avancer |
| **Knowledge Synthesizer** | Synthétiser 3-10 sources en une structure claire | Tu veux maîtriser un sujet à partir de plusieurs sources |
| **Math Rebuild** | Rattrapage maths avec diagnostic + parcours ordonné | Tu as des trous en maths et tu veux les combler |

## Routage par signature

| Signature linguistique | Module |
|---|---|
| *"J'apprends X sur les 3 prochains mois"*, *"je veux maîtriser Y"*, *"par où commencer pour apprendre"* | `learning-path-planner` |
| *"Explique-moi X"*, *"je comprends pas X"*, *"c'est quoi X vraiment"*, *"pourquoi X fonctionne"* | `concept-explainer` |
| *"J'ai 1h pour étudier"*, *"session sur X"*, *"je veux progresser cette semaine sur Y"* | `study-session-designer` |
| *"Synthétise ces sources"*, *"j'ai lu 5 articles, aide-moi à structurer"*, *"vue d'ensemble de X"* | `knowledge-synthesizer` |
| *"Je veux rattraper les maths"*, *"j'ai des trous en maths"*, *"maths pour ML/finance/data"* | `math-rebuild` |

### Détection implicite

Si l'utilisateur :
- Partage un **concept qu'il ne comprend pas** → `concept-explainer`
- Partage **plusieurs sources** (articles, notes, liens) → `knowledge-synthesizer`
- Dit qu'il **bloque depuis longtemps** sur un sujet → commencer par `concept-explainer` sur le point de blocage, puis voir si `math-rebuild` ou `learning-path-planner` est nécessaire

## Composition avec les autres skills

- `learning-os` × `execution-os` : un plan d'apprentissage peut être converti en sprints via `execution-os/sprint-planner`
- `learning-os` × `thinking-os` : pour décomposer un concept complexe, `thinking-os/first-principles` peut être appelé avant `concept-explainer`
- Cas frontière : si l'utilisateur demande *"dois-je apprendre X ou Y ?"* → c'est une décision, rediriger vers `thinking-os/opportunity-cost`

## Procédure d'application

1. **Identifier** le module via la table de routage
2. **Charger** `modules/<module>.md`
3. **Appliquer** rigoureusement la procédure
4. **Produire** la sortie dans le format du module (chaque module a son template dans `templates/`)
5. **Inclure obligatoirement** un mécanisme de vérification (test, livrable, question)
6. **Nommer** le module utilisé en haut de la réponse

## Format de sortie standard

Chaque module a son propre format mais tous respectent ces règles :

- **Pas de résumé sans test.** Chaque output doit inclure un moyen pour l'utilisateur de vérifier qu'il a compris.
- **Livrable observable.** Pas *"étudier X"* mais *"résoudre les 5 exercices du chapitre 3 sans aide"*.
- **Sources externes nommées.** La skill ne prétend pas remplacer les vrais supports (livres, cours, vidéos) — elle les oriente.
- **Artefacts exportables.** Plans, fiches, notes : toujours en markdown structuré que l'utilisateur peut sauvegarder et ré-injecter plus tard.
- **Analogies bornées.** Chaque analogie est accompagnée de ses limites explicites.

## Règles et limites

- **Pas d'apprentissage passif.** La skill refuse de produire de simples résumés sans test de compréhension.
- **Humilité sur les sources externes.** La skill est un accélérateur, pas un substitut à la pratique réelle (exercices, projets, feedback humain).
- **Pas d'évaluation autoritaire.** La skill ne dit pas *"tu n'as pas compris"*. Elle pose une question et laisse l'utilisateur s'évaluer.
- **Respect du niveau.** Si l'utilisateur débute, la skill ne balance pas de jargon technique sans définition.
- **Mémoire assumée.** La skill est consciente qu'elle n'a pas de mémoire persistante entre conversations. Elle produit donc des artefacts que l'utilisateur peut ré-injecter la fois suivante.
- **Pas de fausse promesse.** Maîtriser les maths de prépa en 3 mois en partant de zéro est irréaliste. La skill dira la vérité sur les ordres de grandeur.

## Ressources associées

- `modules/` — 5 modules chargés à la demande
- `templates/` — templates markdown pour les outputs (learning plan, concept card, study session, synthesis, math diagnostic)
- `examples/` — 3 exemples bout-en-bout
- `tests/test-cases.md` — 11 cas pour validation

## Workflow type

```
1. "Je veux apprendre les maths pour ML" → math-rebuild (diagnostic)
2. Le diagnostic révèle des trous en algèbre linéaire → learning-path-planner (parcours 6 semaines)
3. Chaque semaine, 2-3 sessions → study-session-designer
4. Blocage sur "valeurs propres" → concept-explainer
5. Fin du parcours → knowledge-synthesizer (synthèse personnelle des concepts maîtrisés)
```

Cette cadence est une suggestion. La skill s'adapte au rythme réel de l'utilisateur.

---
> Source: [alphilippo/learning-os](https://github.com/alphilippo/learning-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
