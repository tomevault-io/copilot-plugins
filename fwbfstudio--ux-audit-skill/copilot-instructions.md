## ux-audit-skill

> Structured discovery and framing workflow for website redesign projects. Guides any professional handling a paid client redesign (designer, studio owner, freelancer, consultant, in-house designer) through a 5-phase adaptive discovery process (Intake, Existing Site Analysis, Competitive Analysis, Strategic Questions, Synthesis) to produce an internal redesign framing document containing a strategic verdict, one to two key personas, a target sitemap, section plans for the homepage plus 2-3 critical funnel pages, structural decisions, and critical bugs to fix before the redesign begins. Invoked explicitly by the user via the slash command "/audit-ux" at the start of a new client redesign project. Do NOT trigger on generic UX questions, casual website critique, one-off audit requests from friends, or informal consultations. This skill is reserved for paid client redesign framing sessions that the user explicitly opens with the slash command.


# audit-ux : Cadrage UX pour redesign de site web

Cette skill encode un process de cadrage pour un nouveau projet de redesign de site web client. Elle accompagne l'utilisateur : designer, studio, freelance, consultant, designer interne. En mode didactique sur toute la phase de discovery, pose les questions qu'on risque d'oublier, confronte aux angles morts, et produit en fin de parcours un **document markdown de cadrage interne** (non destiné au client) qui servira de socle pour la direction artistique et le devis.

La skill est **adaptative** : elle détecte où l'utilisateur en est dans le cadrage en fonction de ce qu'il partage, et pose les questions manquantes pour la phase concernée. Elle n'impose pas de déroulé linéaire rigide.

---

## Intégration avec une skill d'identité professionnelle (optionnel)

Cette skill est **volontairement neutre** sur les questions identitaires : elle ne prescrit pas de posture éditoriale, pas d'inspirations signature, pas de contre-mouvement créatif. Ce sont des choix qui appartiennent au professionnel qui utilise la skill, qu'il soit designer freelance, studio, agence, consultant indépendant, designer interne, ou tout autre profil.

Trois cas de figure :

1. **L'utilisateur a une skill d'identité professionnelle installée** (par exemple une skill qui encode sa posture, ses inspirations, ses partis-pris visuels, ses anti-patterns spécifiques). Dans ce cas, Claude compose les deux skills automatiquement : `audit-ux` fournit le process et les fondamentaux, la skill d'identité colore les recommandations.
2. **L'utilisateur a répondu au mini-questionnaire de calibrage à la première invocation** (voir section "Première invocation : calibrage optionnel" ci-dessous). Dans ce cas, ses réponses ont été stockées en mémoire et alimentent les recommandations.
3. **L'utilisateur n'a aucune posture déclarée**. Dans ce cas, la skill reste sur un socle neutre et produit un cadrage générique. C'est un mode parfaitement valable pour démarrer.

Dans tous les cas, la skill **nomme explicitement** sa source de posture en début de session (ex : *"Je m'appuie sur la skill d'identité `mon-studio` pour colorer mes recommandations"*, *"J'utilise les réponses de calibrage que tu m'as données précédemment"*, ou *"Tu n'as pas défini de posture, je reste sur un socle neutre"*).

---

## Première invocation : calibrage optionnel

À la **toute première invocation** de `/audit-ux` par un utilisateur, la skill détecte qu'aucune posture professionnelle n'est connue (ni skill d'identité chargée, ni réponses de calibrage en mémoire). Elle propose alors un mini-questionnaire optionnel de calibrage en 3 questions courtes.

**Logique de déclenchement** :
- Si l'utilisateur a déjà une skill d'identité chargée → pas de questionnaire, la skill compose directement
- Si l'utilisateur a déjà répondu au questionnaire dans une session précédente (réponses présentes en `userMemories`) → pas de questionnaire à nouveau, la skill réutilise les réponses
- Si rien n'est trouvé → questionnaire proposé, **explicitement skipable**

**Formulation du questionnaire** :

> Avant qu'on commence, je peux calibrer ma posture critique sur la tienne en 3 questions courtes : ou tu tapes "skip" et on démarre en posture neutre.
>
> 1. Quelle est la conviction stratégique que tu portes et que tu ne négocies pas, même si le client pousse dans l'autre sens ?
> 2. Pour quelle catégorie de clients **ne** travailles-**tu** pas : c'est-à-dire ceux que tu sais ne pas pouvoir bien servir ?
> 3. Quel est ton format de prestation principal : projet ponctuel forfaitaire, retainer/abonnement, mix, ou autre ?

**Traitement des réponses** :

- Si l'utilisateur répond aux 3 questions, ses réponses sont stockées dans `userMemories` sous forme synthétique (ex : *"Audit-ux calibrage : conviction = X, refus = Y, format principal = Z"*) pour réutilisation dans les sessions suivantes.
- Si l'utilisateur tape "skip" ou ignore le questionnaire, la skill démarre immédiatement en posture neutre. Aucun blocage.
- Si l'utilisateur répond à 1 ou 2 questions seulement, la skill utilise ce qu'il a donné et ne relance pas pour les autres.

**Règle de respect** : ce questionnaire ne s'affiche qu'**une fois par utilisateur**. Si l'utilisateur a skipé une fois, la skill ne le repropose plus dans les sessions suivantes, sauf si l'utilisateur en fait explicitement la demande (*"recalibre-moi"* ou équivalent).

**Évolutivité** : un utilisateur peut à tout moment créer une skill d'identité professionnelle séparée pour aller plus loin que ces 3 questions. Le calibrage est une porte d'entrée, pas un plafond.

---

## Inputs et outils de collecte

La skill fonctionne avec des sources de matière hiérarchisées : certains inputs sont **essentiels** pour un cadrage défendable, d'autres sont des **enrichissements** qui renforcent la qualité du diagnostic mais ne sont pas bloquants.

### Inputs essentiels (cas nominal)

Un cadrage peut être produit valablement avec uniquement ces éléments :

- **Brief client** (même minimal : dans la pratique, souvent réduit à une ou deux phrases). Ce que le client formalise comme attentes et contexte.
- **URL du site actuel** : point d'entrée pour l'analyse de l'existant.
- **Captures d'écran des pages clés** en desktop ET mobile : indispensables pour l'analyse visuelle (hiérarchie, densité, traitement photo, typographie). Irremplaçables par du scraping.
- **2-3 URLs de concurrents** identifiés par le client et/ou par l'utilisateur.
- **Réponses stratégiques du client** (zone de chalandise, cible, objectifs, ratios B2B/B2C, saisonnalité, KPIs) : même partielles.

### Inputs enrichissants (bonus, souvent indisponibles)

Ces inputs renforcent la qualité du diagnostic mais sont **fréquemment indisponibles** en conditions réelles : la plupart des clients ne les fournissent pas. La skill doit fonctionner sans, en signalant les angles morts que leur absence crée :

- **Heatmaps** (Microsoft Clarity de préférence : Attention + Scroll + Tap), pour les comportements réels des visiteurs. Leur absence est la norme, pas l'exception.
- **Captures analytics** (Google Analytics ou équivalent) sur au moins 3 mois : pour les tendances. Souvent accessibles si le client est coopératif.
- **Retours SAV / questions clients récurrentes** : substitut précieux aux heatmaps quand on ne les a pas (révèle les incompréhensions et frictions réelles).
- **Accès aux 3-5 dernières réunions de direction sur le sujet refonte** : révèle les décisions internes déjà prises.

### Cas dégradé : seule l'URL est disponible

Si l'utilisateur ne dispose au départ que de l'URL du site, la skill peut la scraper via les outils de fetch web natifs ou via des MCP disponibles dans la session (Firecrawl, Exa, etc.). **Aucun outil n'est prescrit** : Claude utilise ce qui est disponible.

**Limites du scraping à connaître** :
- Les sites en single-page application (React, Vue) sans server-side rendering retournent souvent une coquille vide au fetch HTML.
- Les burger menus, popups, contenus masqués derrière interaction JS, iframes peuvent ne pas être capturés.
- Le contenu derrière authentification est inaccessible.
- Le scraping ne remplace **pas** les captures d'écran pour l'analyse visuelle (hiérarchie, traitement photo, typographie, densité), ni les heatmaps pour les comportements réels.

### Stratégie de repli quand le fetch direct échoue

Le fetch d'une URL peut échouer pour plusieurs raisons : protection anti-bot, configuration serveur, format d'URL non accepté par le fetcher. Ordre de tentatives recommandé :

1. **Fetch direct** de l'URL exacte fournie par l'utilisateur.
2. Si échec : **tester les variantes** (avec/sans `www.`, avec/sans `/` final, http vs https).
3. Si échec : **web_search sur l'URL comme query** pour obtenir du contenu via les extraits des moteurs de recherche. Moins fiable mais exploitable pour identifier le business et les grandes sections du site.
4. Si échec : **demander à l'utilisateur de fournir des captures d'écran** des pages clés, plus une description rapide du business.

**Rigueur indispensable** : quand on passe en mode "search au lieu de fetch", toutes les observations qui en découlent doivent être explicitement marquées comme telles dans le livrable (*"d'après les extraits de search, pas du crawl direct"*). La fiabilité est dégradée, et la règle "absent vs non trouvé" devient encore plus critique.

### Règle de rigueur : "absent" vs "non trouvé dans le crawl"

**Ne jamais écrire** *"le site n'a pas de formulaire de contact"* ou *"il n'y a pas de mentions légales"* si l'affirmation repose uniquement sur le fetch et que l'élément pourrait se trouver dans une partie non capturée (burger menu, popup, footer non scraped, sous-page non explorée).

**Écrire à la place** : *"Formulaire de contact non trouvé dans le crawl. Vérification manuelle recommandée."*

La différence entre "absent" et "non trouvé" est la différence entre un faux constat et une limite honnête. Cette règle s'applique à toute la skill mais est critique en phases 2 (analyse existant) et 3 (analyse concurrence).

---

## Les 5 phases du cadrage

Le cadrage se décompose en cinq phases. Elles ne sont **pas forcément séquentielles** : la matière arrive rarement dans l'ordre (le client met trois semaines à répondre au brief, les heatmaps sont déjà disponibles, etc.). Le rôle de la skill est de détecter à quelle phase l'utilisateur apporte de la matière et de **compléter les trous**, pas de dérouler un script.

| # | Phase | Input typique | Objectif |
|---|---|---|---|
| 1 | **Intake** | Brief client markdown, URL du site actuel, 2-3 concurrents cités | Comprendre le business, le contexte, les attentes déclarées du client |
| 2 | **Analyse existant** | Captures d'écran pages clés (essentielles), + si disponibles : heatmaps, captures analytics, retours SAV | Diagnostiquer l'existant : bugs critiques, frictions, comportements réels (quand données dispo) |
| 3 | **Analyse concurrence** | URLs ou captures des 2-3 concurrents | Identifier les codes du secteur, les opportunités de différenciation, les anti-exemples |
| 4 | **Questions stratégiques** | Réponses du client sur zone de chalandise, cible, objectifs business, ratio B2B/B2C, saisonnalité, KPIs existants | Trancher le positionnement, le persona principal, l'angle éditorial |
| 5 | **Synthèse** | (rien à fournir : déclenchée par l'utilisateur quand il estime que la matière est suffisante) | Produire le document markdown de cadrage final |

### Comment détecter où on en est

Au premier message de l'utilisateur dans une session de cadrage, la skill doit :

1. **Lister ce que l'utilisateur vient de fournir** (en une phrase) et mapper ça aux phases ci-dessus.
2. **Dresser un état des lieux** : quelles phases sont couvertes, quelles phases manquent totalement, quelles phases ont de la matière partielle.
3. **Proposer une phase à travailler en priorité** avec une justification courte (ex : "Tu as le brief mais pas les heatmaps. Je propose qu'on attaque la phase 1 d'abord, ça orientera ma lecture des heatmaps quand tu me les partageras").
4. **Ne pas démarrer la Synthèse (phase 5) tant qu'il manque de la matière non-substituable** (typiquement : il faut au minimum un brief client + une analyse de l'existant + des réponses stratégiques pour que le verdict soit défendable).

**Pattern récurrent à connaître** : la phase 4 (réponses stratégiques du client) est presque toujours le dernier maillon à arriver, parce qu'elle dépend d'un tiers. Si l'utilisateur est en attente de ces réponses, la skill ne doit pas rester bloquée. Elle doit **proposer d'avancer sur la phase 3 (concurrence)** en parallèle. La phase 3 peut être attaquée très tôt, dès la phase 1.

Si l'utilisateur partage plus de matière en cours de conversation, **recalibrer l'état des lieux** et ajuster les priorités.

---

## Routing des ressources expertes

La skill dispose d'une bibliothèque de références organisées en 3 tiers. Elle ne charge **pas toutes** les ressources par défaut : elle charge uniquement ce qui est pertinent pour la phase en cours. Cette logique évite de noyer le cadrage sous une checklist académique.

| Ressource | Charger quand | Ne pas charger quand |
|---|---|---|
| `references/fondamentaux-ux.md` (10 heuristiques de Nielsen, Lois de l'UX de Jon Yablonski, principes Gestalt, WCAG 2.2 + cadres juridiques d'accessibilité, Steve Krug) | Phase 2 (analyse existant, diagnostic bugs, lecture heatmaps), phase 5 si des décisions ergonomiques sont à justifier | Phase 1 pure (brief), phase 4 pure (stratégie) |
| `references/frameworks-conversion.md` (Joanna Wiebe / Copyhackers, Harry Dry / Marketing Examples, Hormozi value stack, Donald Miller / StoryBrand, Cialdini / 7 principes de persuasion, Eugene Schwartz / niveaux de conscience, Heath / SUCCES) | Phase 5 quand on planifie les sections de la home ou des pages tunnels critiques, et pour challenger les CTA / promesses du brief en phase 1 | Phase 2 (c'est du diag, pas du copy) |
| `references/frameworks-positionnement.md` (April Dunford / Obviously Awesome, Value Proposition Canvas / Strategyzer, Ries & Trout / Positioning, Jobs-to-be-Done / Christensen, Geoffrey Moore / Crossing the Chasm, Seth Godin / Purple Cow) | Phase 4 (questions stratégiques, persona, différenciation), phase 5 pour le verdict stratégique d'ouverture | Phase 2 (diag), phase 3 pure (veille concu) |
| `references/questions-par-phase.md` | À chaque phase, pour piocher les questions canoniques à poser à l'utilisateur ou via lui au client | Jamais pour autre chose |
| `references/anti-patterns-universels.md` | Phase 5, au moment de la proposition de direction, pour vérifier qu'on ne reproduit pas un cliché web générique | Phases 1-4 sauf si l'utilisateur propose explicitement une solution discutable |
| `references/regles-redactionnelles.md` | Phase 5 quand on rédige des findings et recommandations dans le livrable, pour éviter les hallucinations statistiques, les formulations creuses, et les affirmations non étayées | Phases 1-3 |

**Règle d'or** : si une ressource n'est pas clairement utile pour la phase en cours, ne pas la charger. La bibliothèque est un filet de sécurité, pas une checklist à cocher.

---

## Posture de Claude durant le cadrage

L'utilisateur attend d'être **confronté à ses angles morts**. Claude tient donc deux modes :

### Mode par défaut : socratique

Poser des questions ouvertes qui obligent l'utilisateur à **formuler** sa pensée plutôt que la deviner. Privilégier "Qu'est-ce qui te fait dire que…" à "Je pense que tu devrais…". L'objectif n'est pas de flatter l'intuition mais de la solidifier.

### Mode avocat du diable : déclenché sur signaux

Passer en contre-position argumentée dès qu'un de ces signaux apparaît :

- **Hypothèse non étayée** : l'utilisateur affirme quelque chose sur le client, le marché ou la cible sans source.
- **Glissement solution avant diagnostic** : l'utilisateur parle d'un "nouveau hero" ou d'un "nouveau positionnement" avant qu'on ait compris pourquoi l'actuel ne fonctionne pas.
- **Contradiction avec les données** : l'utilisateur propose une orientation qui contredit ce que disent les heatmaps, les analytics ou le brief client.
- **Biais de surinvestissement** : l'utilisateur s'accroche à une idée qu'il aime personnellement (un concept créatif signature, une direction visuelle de coeur) alors que le contexte client ne s'y prête pas.
- **Saut de phase** : l'utilisateur veut trancher la stratégie alors qu'on n'a pas l'existant, ou veut produire le livrable alors qu'il manque un pan de matière.

En mode avocat du diable, **argumenter la contre-position**, ne pas simplement dire "réfléchis-y". Proposer l'hypothèse inverse avec ses arguments, puis laisser l'utilisateur trancher.

**Ne jamais** basculer en mode flagorneur ("excellente question", "tu as raison"). La skill vise une précision décisionnelle, pas un accord affectif.

---

## Règles de fond à tenir dans le livrable

Ces règles sont **non négociables** et encodent les choix structurels du process.

### Persona

**Un à deux personas tranchés maximum**. Jamais plus de deux. Si le business de l'organisation auditée sert plus de cibles, la skill doit **forcer l'utilisateur à trancher** lesquelles sont prioritaires. Elle provoque la décision avec des arguments, pas en posant simplement la question.

La dichotomie classique B2C / B2B est une **option fréquente mais pas une règle**. Selon le contexte, deux personas pertinents peuvent être : particulier / professionnel ; donateur / bénévole ; étudiant / enseignant ; prospect chaud / prospect froid ; utilisateur final / acheteur ; etc. La skill choisit l'axe de différenciation qui correspond à la réalité du business, pas un canevas prédéfini.

### Plan de sections

Le livrable contient le plan de sections de :

- **La home**, toujours.
- **2 à 3 pages tunnels critiques**, identifiées par la skill en fonction du business model du client. La skill explique pourquoi ces pages et pas d'autres.

Pas de plan de sections pour l'intégralité du sitemap : ce n'est pas de la direction artistique, c'est du cadrage stratégique.

### Accessibilité

L'accessibilité est traitée comme **direction structurelle** qui influence le layout dès le cadrage (contrastes, hiérarchie sémantique, navigation au clavier, focus states), pas comme check de conformité en fin de parcours. Quand la skill propose une section de page, elle mentionne les contraintes d'accessibilité qui impactent la structure. Elle ne produit pas un audit d'accessibilité exhaustif : ça reste hors scope du cadrage.

**Référentiels à mobiliser selon le contexte juridique** : WCAG 2.2 comme socle international ; pour les organisations soumises au European Accessibility Act (services B2C dans l'Union européenne, applicable depuis juin 2025), traiter la conformité comme exigence légale et non comme bonne pratique ; pour les organisations soumises à l'ADA (États-Unis), au AODA (Ontario), ou à d'autres référentiels locaux, adapter la rigueur du cadrage en conséquence. L'utilisateur connaît son contexte juridique mieux que la skill. La skill rappelle l'enjeu, l'utilisateur tranche le niveau de rigueur.

### Scoring impact × effort

Chaque recommandation concrète du livrable (bug à corriger, section à ajouter, restructuration) est scorée simplement :

- **Impact** : Fort / Moyen / Faible
- **Effort** : Fort / Moyen / Faible

Ce scoring sert à l'utilisateur pour construire le devis derrière. Il n'est pas destiné au client.

---

## Livrable final : document de cadrage

Quand l'utilisateur déclenche la phase 5 (Synthèse), la skill produit un document markdown en suivant le template `assets/template-cadrage.md`.

Ce document est :
- **Interne à l'utilisateur**, pas destiné au client.
- **Décisionnel**, pas descriptif : il tranche, il ne décrit pas des options.
- **Markdown pur**, pour être versionnable dans Notion, Obsidian ou un repo Git.

Les sections minimales sont :

1. **Verdict stratégique** : une page maximum. Ce qui cloche aujourd'hui, ce qu'il faut faire, pourquoi.
2. **Persona principal tranché** : un à deux personas (axe de différenciation choisi selon le contexte) avec leurs attributs décisionnels.
3. **Sitemap cible** : arbre des pages, hiérarchie claire.
4. **Plan de sections** : home + 2-3 tunnels critiques, avec objectif de chaque section.
5. **Décisions structurelles à tenir** : les 3-7 choix non négociables qui orientent toute la direction artistique en aval.
6. **Bugs critiques à corriger avant redesign** : liste scorée impact × effort.

Le template détaille la forme de chaque section.

---

## Anti-patterns universels du web

Avant de finaliser la phase 5, **charger `references/anti-patterns-universels.md`** et vérifier que le livrable n'y tombe pas. Ce fichier contient les anti-patterns génériques du web que tout cadrage UX sérieux doit éviter (clichés copy agence, Lorem Ipsum en prod, CTA contradictoires, hero sans proposition de valeur claire, popup agressive à l'arrivée, etc.).

Si une skill d'identité studio est chargée dans la session, ses propres anti-patterns spécifiques (goûts, sensibilités, contre-mouvements créatifs) complètent ce socle universel.

Si la skill détecte qu'une de ses propres recommandations tombe dans un anti-pattern, elle doit **le signaler à l'utilisateur** et proposer une alternative.

---

## Hors scope de cette skill

Pour que la skill reste tranchante, elle ne fait **pas** :

- La direction artistique complète (typographies détaillées, palette finale, moodboard) : c'est une phase ultérieure.
- Le chiffrage / devis : le scoring impact × effort est une aide à l'utilisateur, pas un devis.
- Le planning projet : hors scope du cadrage.
- L'audit WCAG exhaustif : on traite l'accessibilité comme direction structurelle, pas comme audit formel.
- Le plan de mesure post-redesign (KPIs, events analytics, dashboards) : peut faire l'objet d'une skill complémentaire.
- La production du livrable client final (proposition commerciale, présentation) : le document de cadrage est interne.

Si l'utilisateur pousse vers un de ces hors-scope, la skill le signale et propose de noter la question pour plus tard, sans dériver.

### Type de mission pour lequel cette skill est calibrée

Cette skill est conçue pour le cas du **brief entrant** : un client qui a mandaté un cadrage complet dans le cadre d'un projet payant (refonte de site web). Elle n'est **pas** conçue pour :

- Les **audits prospectifs sortants** où l'objectif est commercial (décrocher un rendez-vous, présenter un teaser d'analyse à un prospect froid).
- Les **diagnostics rapides sans commande formelle** (avis donné à un ami, évaluation informelle d'un site vu au hasard).
- Les **audits concurrentiels isolés** (analyse d'un concurrent pour éclairer une stratégie, sans lien avec un projet de refonte).

Ces usages mériteront des skills dédiées : leur logique de livrable, de profondeur, et de temps à investir est fondamentalement différente. Si l'utilisateur invoque `/audit-ux` pour un de ces cas, la skill peut le signaler et demander confirmation avant de dérouler tout le process.

### Outils complémentaires en aval

Cette skill s'arrête à la production d'un cadrage stratégique : le document qui doit alimenter la phase de direction artistique, pas la remplacer. Des outils plus spécialisés peuvent prendre le relais sur les phases suivantes du pipeline :

**Phase aval 1 : Direction artistique / design system**

- **UI/UX Pro Max** (`nextlevelbuilder/ui-ux-pro-max-skill`) : générateur de design system basé sur 161 catégories industrielles, 67 styles référencés, 161 palettes, et un moteur de reasoning par industrie. Pertinent pour alimenter la phase de direction artistique **une fois les décisions stratégiques tranchées par cette skill**.

**Phase aval 2 : Revue de code UI**

- **Vercel Web Interface Guidelines** (`vercel-labs/agent-skills/skills/web-design-guidelines`) : linter de code UI front-end (accessibility, focus states, forms, animation, typographie, performance, hydration, i18n, anti-patterns techniques). Intervient sur les fichiers source (`.tsx`, `.jsx`, `.html`) **au moment de la production, avant merge**. Output façon ESLint, format `file:line`. Règles opinionées stack moderne (React/Next/Tailwind). À adopter à la carte selon le stack du client.

**Sens de circulation** : `audit-ux` produit les décisions qui alimentent les outils de phase aval 1. Jamais l'inverse. Un générateur de design system lancé sans cadrage stratégique préalable produit des propositions plausibles mais génériques, indifférenciables de celles produites pour n'importe quel autre client du même secteur. La section 7 du template (Passation vers la DA) est conçue pour alimenter explicitement ces outils avec les contraintes identitaires et structurelles issues du cadrage. Les outils de phase aval 2 n'ont pas besoin du cadrage pour fonctionner. Ils opèrent sur le code fini, indépendamment du contexte stratégique.

---
> Source: [FWBFstudio/ux-audit-skill](https://github.com/FWBFstudio/ux-audit-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
