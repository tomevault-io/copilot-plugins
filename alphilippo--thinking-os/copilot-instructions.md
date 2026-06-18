## thinking-os

> Routeur cognitif qui détecte automatiquement le type de problème posé (décision, arbitrage, décomposition, diagnostic, compréhension) puis applique le bon framework de pensée parmi une bibliothèque de 12 modèles (first principles, second-order thinking, inversion, OODA, pre-mortem, 5 whys, expected value, opportunity cost, Bayesian update, circle of competence, Eisenhower, 10/10/10). Utilise cette skill dès que l'utilisateur pose une question contenant "est-ce que je devrais", "j'hésite entre", "pourquoi", "comment décider", "c'est trop complexe", "j'arrive pas à trancher", "utilise first principles/inversion/OODA/etc. sur X", ou toute question stratégique, décisionnelle ou de diagnostic. Déclenche aussi quand l'utilisateur demande explicitement un framework par son nom, ou quand la question est clairement une réflexion de fond plutôt qu'une demande d'information. Produit par défaut une synthèse courte orientée action, avec deep-dive détaillé sur demande ("détaille", "explique plus", "deep-dive").


# thinking-os

Un **routeur cognitif** pour appliquer le bon framework de pensée à chaque type de problème.

## Philosophie

Les gens collectionnent des mental models comme des timbres, puis n'en appliquent aucun correctement. Cette skill fait l'inverse : elle **diagnostique d'abord le type de problème**, puis **applique rigoureusement UN framework adapté** — au lieu de balancer des généralités.

Un bon cadre appliqué systématiquement > dix cadres évoqués en passant.

## Modes de déclenchement

### Mode auto (par défaut)
Si l'utilisateur ne nomme pas de framework, suis le **routeur** ci-dessous pour en choisir un.

### Mode explicite
Si l'utilisateur dit *"utilise first principles sur X"*, *"applique inversion à Y"*, *"fais-moi un pre-mortem"* — saute le routeur et va directement au fichier `frameworks/<framework>.md`.

Frameworks explicitement adressables :
`first-principles`, `second-order`, `inversion`, `ooda-loop`, `pre-mortem`, `5-whys`, `expected-value`, `opportunity-cost`, `bayesian`, `circle-of-competence`, `eisenhower`, `10-10-10`

---

## Le routeur cognitif

Détecter la **signature** du problème posé, puis matcher au framework.

| Signature du problème | Indice linguistique | Framework à charger |
|---|---|---|
| Remise à plat d'un sujet | *"pourquoi on fait ça ?"*, *"c'est quoi vraiment X ?"*, *"tout le monde dit Y mais..."* | `first-principles` |
| Décision avec conséquences en cascade | *"si je fais X, ensuite..."*, *"à long terme"*, *"les effets de second ordre"* | `second-order` |
| Comment éviter un échec | *"qu'est-ce qui pourrait foirer"*, *"comment ne pas rater"*, *"risques"* | `inversion` |
| Contexte compétitif/rapide | *"le concurrent vient de..."*, *"il faut réagir"*, *"situation bouge vite"* | `ooda-loop` |
| Avant un lancement/décision engageante | *"je vais lancer X"*, *"avant de signer"*, *"avant de committer"* | `pre-mortem` |
| Diagnostic de cause racine | *"ça marche pas, pourquoi ?"*, *"problème récurrent"*, *"toujours le même bug"* | `5-whys` |
| Arbitrage avec probabilités | *"quelle chance que ça marche"*, *"vaut mieux X ou Y"* avec incertitude | `expected-value` |
| Arbitrage entre options concrètes | *"j'hésite entre A et B"*, *"option 1 ou option 2"* | `opportunity-cost` |
| Intégrer une info nouvelle | *"je viens d'apprendre X, ça change quoi ?"*, *"nouvelle data"* | `bayesian` |
| Suis-je qualifié pour décider ? | *"je comprends à moitié"*, *"domaine que je maîtrise pas"* | `circle-of-competence` |
| Priorisation urgent vs important | *"trop de choses à faire"*, *"par où commencer"*, *"tout est urgent"* | `eisenhower` |
| Décision émotionnelle | *"je suis en colère"*, *"j'en ai marre"*, *"sur le coup je dirais X"* | `10-10-10` |

### Si plusieurs frameworks matchent

Ordre de priorité :
1. Framework explicitement nommé par l'utilisateur > tout le reste
2. Signature émotionnelle (`10-10-10`, `circle-of-competence`) > autres (pour protéger la qualité de la décision)
3. Signature compétitive/temps court (`ooda-loop`) > analyse longue
4. Si vraiment ambigu : applique **first-principles** par défaut (c'est le framework le plus général)

### Si aucun framework ne matche

La question n'est probablement pas une réflexion de fond — c'est une demande d'info. Ne pas activer la skill. Répondre normalement.

---

## Procédure d'application

Une fois le framework sélectionné :

1. **Charger** le fichier correspondant : `frameworks/<framework>.md`
2. **Appliquer** rigoureusement la procédure décrite dans ce fichier au contexte spécifique de l'utilisateur
3. **Produire la sortie** selon le format demandé :
   - **Par défaut** : synthèse courte + action claire (voir `frameworks/<framework>.md` section "Format synthèse")
   - **Si l'utilisateur demande "détaille" / "deep-dive" / "explique plus"** : analyse structurée complète (voir section "Format deep-dive")
4. **Nommer explicitement le framework utilisé** en haut de la réponse — *"Je traite ça en first-principles"* — pour que l'utilisateur apprenne aussi la méthode.

---

## Format de sortie standard

**Ne jamais** retourner juste une liste de mental models. **Toujours** appliquer concrètement au cas posé.

### Format synthèse (par défaut)

```
🧠 Framework : <nom>
📌 Pourquoi ce cadre : <1 ligne>

<application concrète du framework au problème, 3-6 lignes>

✅ Action : <UNE action claire>
⚠️ Angle mort : <UN risque à surveiller>
```

### Format deep-dive (sur demande)

Structure adaptée au framework, typiquement 300-500 mots :
- Reformulation du problème
- Application étape par étape du framework
- Alternatives considérées et rejetées
- Limites de l'analyse
- Action recommandée + métriques pour valider

Voir chaque fichier `frameworks/<name>.md` pour la structure exacte de deep-dive.

---

## Règles et limites

- **Ne pas empiler les frameworks.** Un seul à la fois, sauf demande explicite de combinaison.
- **Ne pas philosopher.** L'utilisateur veut une réponse actionnable, pas un cours de Charlie Munger.
- **Pas de name-dropping.** Ne pas citer "comme disait Buffett..." sauf si c'est vraiment éclairant.
- **Respect du contexte.** Si l'utilisateur a déjà choisi, ne pas remettre la décision en cause — aide à exécuter.
- **Détection d'émotion prioritaire.** Si la question contient des marqueurs émotionnels forts (*"marre", "en colère", "épuisé"*), commencer par `10-10-10` ou `circle-of-competence` avant tout autre cadre.
- **Pas de conseil financier, juridique ou médical définitif.** Les frameworks aident à structurer la pensée, pas à remplacer un expert.

---

## Ressources associées

- `frameworks/` — un fichier par framework, chargé à la demande
- `examples/` — 3 exemples de bout-en-bout pour calibrer le ton
- `tests/test-cases.md` — 10 cas pour valider la skill après édition
- `README.md` — doc user-facing si publication GitHub

---
> Source: [alphilippo/thinking-os](https://github.com/alphilippo/thinking-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
