## quiz-funnel-expert

> Expert end-to-end pour concevoir, auditer et coder des quiz funnels haute-conversion (mobile apps + SaaS web) couplés à des paywalls optimisés. Couvre la stratégie (croyances → questions → personnalisation → paywall), le copywriting des questions et résultats, l'intégration paywall (hard/soft, trial mechanics, RevenueCat/Superwall/Adapty patterns), les benchmarks 2026 (RevenueCat 115k apps, Adapty 16k apps), 30+ teardowns (Cal AI, Noom, Duolingo, Blinkist, Flo, Opal, Stoic, Calm, Headspace, MyFitnessPal, Linear, Notion, Anthropic, Cursor, Superhuman), le A/B test playbook par win-rate, l'instrumentation analytics, la stack technique (Next.js + Zustand + Stripe/RevenueCat/Superwall) ET un sous-skill "Dashboard Réponse" pour générer un back-office admin complet (KPI, funnel par étape, drop-off, distribution réponses, détail session, export CSV) schema-agnostic et production-ready. Use when the user asks for "quiz funnel", "onboarding quiz", "tunnel de quiz", "quiz d'audit", "quiz personnalisation", "questionnaire de conversion", "paywall", "tunnel d'abonnement", "subscription funnel", "trial conversion", "freemium vs hard paywall", "Noom-style funnel", "web-to-app funnel", "personalized paywall", "quiz to paywall handoff", "augmenter mon trial-to-paid", "concevoir un onboarding mobile", "pricing page SaaS", "review trial mechanics", "dashboard réponse", "back-office quiz", "admin quiz funnel", "voir les réponses du quiz", "drop-off rate par étape", "completion rate quiz", "analytics interne quiz", or names a benchmark app (Cal AI, Noom, BetterMe, Flo, Blinkist, Calm, Headspace, Duolingo, Strava, MyFitnessPal, Linear, Notion, Anthropic, Cursor, Superhuman, Vercel, Stripe, Raycast).


# Quiz Funnel Expert

Skill expert pour concevoir, auditer, coder et optimiser un **quiz funnel + paywall** haute-conversion. Couvre stratégie, copy, personnalisation, intégration paywall, benchmarks 2026, A/B test playbook et stack technique.

Le quiz funnel est **l'arme #1 de conversion mobile/SaaS 2024-2026**. Cal AI ($30M ARR), Noom ($750M ARR), Flo ($9M/mois), Cal AI, Opal, Stoic, BetterMe, Blinkist tournent tous sur ce pattern : **quiz → personalized plan → paywall**. Ce skill encode ce qui marche.

---

## Quand utiliser ce skill

Trigger sur :
- "Concevoir un quiz funnel / onboarding quiz / tunnel d'abonnement"
- "Optimiser un paywall (mobile app, SaaS web, freemium, trial)"
- "Auditer un onboarding existant"
- "Personnaliser un paywall selon les réponses utilisateur"
- "Choisir hard paywall vs freemium vs soft paywall"
- "Définir trial mechanics (3j vs 7j vs 14j vs reverse trial)"
- "Faire un teardown style Noom / Cal AI / Blinkist"
- "Coder un quiz funnel Next.js"
- "Pricing page SaaS (Linear, Notion, Anthropic style)"
- **Sous-skill Dashboard Réponse** : "back-office quiz", "dashboard admin quiz", "voir les réponses", "voir les leads / sessions", "drop-off rate par étape", "completion rate", "distribution des réponses par question", "export CSV des leads", "page admin pour une session" → router vers [12-admin-dashboard.md](references/12-admin-dashboard.md).

NE PAS trigger pour :
- Onboarding produit B2B post-signup pure (utiliser `onboarding-cro`)
- Cancel flow / churn save flows (utiliser `churn-prevention`)
- Public landing page sans quiz (utiliser `landing-page-expert`)

---

## Thèse centrale (à imprimer dans la mémoire de l'agent)

1. **L'onboarding est une conversion de croyance, pas un tour de produit.** Chaque écran doit déplacer l'utilisateur d'un état "incertitude" vers "ce produit comprend mon cas, peut me donner un résultat, vaut le prochain engagement".
2. **Le questionnaire est un asset de conversion uniquement s'il produit une personnalisation visible.** Si une réponse ne change ni le plan, ni le paywall, ni le segment, ni l'expérience → la question doit disparaître.
3. **Les 5 écrans avant le paywall comptent plus que le paywall lui-même** (Stormy, 4500+ A/B tests). Onboarding + paywall = un seul funnel.
4. **Day 0 est tout** : 90% des trial starts arrivent J0 (Adapty 2026), 84% des cancel arrivent J0-1 sur trial 3j (RevenueCat 2026). Si l'aha n'est pas dans la première session, c'est mort.
5. **Hard paywall n'a pas de pénalité de rétention** vs freemium (RevenueCat 2026, 115k apps : Y1 retention ~27% vs ~28%). La folklore "freemium retains better" est morte. Choix = fonction de CAC, pas d'idéologie.
6. **Long quiz convertit IF chaque question paie son loyer.** Noom = 113 écrans qui marchent. La majorité des quiz longs sont juste de la friction.

---

## Workflow par défaut quand on invoque ce skill

### Étape 1 — Diagnostic

Avant tout output, comprendre :

1. **Modèle économique** : mobile subscription / SaaS B2B / SaaS B2C / hybrid web-to-app / one-time purchase ?
2. **Plateforme** : iOS only / Android only / hybrid / web only ?
3. **CAC & LTV target** : si CAC > 30€ → hard paywall probablement ; si CAC < 5€ et viralité → soft/freemium.
4. **Catégorie** : Health & Fitness / Education / Productivity / AI / Finance / Lifestyle / Dating / SaaS dev tools / SaaS PLG / Enterprise sales-led ? Les benchmarks et patterns dominants varient.
5. **Aha moment** : quelle est l'action qui corrèle le plus avec la rétention J7 ? (À faire trouver à l'utilisateur si pas clair.)
6. **État actuel** : zéro / quiz existant à auditer / paywall existant à optimiser / cold redesign ?
7. **Contraintes Apple/Google** : hard paywall iOS Jan 2026 = toggle paywall mort (rejet Guideline 3.1.2), close button obligatoire, etc. Voir [reference 04](references/04-paywall-integration.md).

Si plusieurs infos manquent et bloquent → **demander à l'utilisateur en un seul tour groupé** (max 3-5 questions). Sinon avancer avec assumptions explicitées.

### Étape 2 — Choix structurels (avant copy)

Décider dans cet ordre, en s'appuyant sur les références :

| Décision | Référence | Default 2026 |
|---|---|---|
| Hard / soft / freemium / reverse trial | [05](references/05-paywall-benchmarks-2026.md) §Hard vs soft | **Hard pour mobile B2C subscription si CAC justifie ; reverse trial pour SaaS PLG mature ; freemium pour produit réseau/social** |
| Quiz length | [02](references/02-questionnaire-design.md) §Long vs court | 8-12 questions Health/Education ; 4-6 utilities ; 30+ Noom-style si justifié |
| Paywall placement | [04](references/04-paywall-integration.md) §Placement | Après plan personnalisé (pattern dominant) ; FitnessAI/Rootd ont +2x install-to-trial avec paywall **avant** onboarding (à tester) |
| Trial duration | [04](references/04-paywall-integration.md) §Trial mechanics | 3j (Adapty top config weekly+3d trial) ou 7j (sweet spot général) ; 14-30j Productivity/Business |
| Plans visibles | [05](references/05-paywall-benchmarks-2026.md) §Number of plans | **Single plan visible + "View all plans" link** dépasse souvent 3-plan en mobile ; SaaS = 3 self-serve + Enterprise |
| Default plan | — | **Annual préselectionné** → 2-3x annual mix, +70% annual revenue (Sunflower/Stormy) |
| CTA wording | [08](references/08-copy-library.md) | "Continue" beat descriptif +111% (Stormy) ; "Start Free Trial" / "Try for $0.00" / "Get [Plan Name]" |
| Pricing display | [05](references/05-paywall-benchmarks-2026.md) §Price framing | Per-week framing ($39.99/yr → "$0.76/week") + "SAVE 75%" anchor ; rond pour SaaS premium ($20, $30) |
| Web-to-app vs IAP | [04](references/04-paywall-integration.md) §Web-to-app | **Hybrid IAP+web** sur iOS US (web-only perd 6.5% net vs IAP) ; web-to-app standard pour Health/Education/AI haute LTV |

### Étape 3 — Architecture du flow (livrable principal)

Map le funnel screen-by-screen avec, pour CHAQUE écran :

- **Objectif/croyance convertie** (ex. "L'app comprend mon contexte")
- **Type d'écran** (entry / question / micro-insight / loading / result / paywall / commit)
- **Question + 4-5 options** (si Q) ou **copy headline + sub** (si insight/result/paywall)
- **Personnalisation triggered** : quelle variable ce screen alimente downstream
- **Drop-off risk** + mitigation
- **Event tracking** (voir [11](references/11-metrics-and-tracking.md))

Modèle de flow gagnant (à adapter) :

```
[Entry promise: result-focused, 1 écran max]
→ Goal question (objectif principal)
→ Current situation (état actuel)
→ Friction (blocage principal)
→ MICRO-INSIGHT #1 (utiliser réponses 1-3 → "On va adapter ton plan pour…")
→ Level/context question
→ Preference/frequency
→ MICRO-INSIGHT #2 (social proof segmenté : "Les [segment] obtiennent +X en Y semaines")
→ Commitment question (engagement léger)
→ Loading screen (3-7s, "On construit ton plan…")
→ Personalized result/preview (score + plan + timeline)
→ [Optional: First tiny action / preview]
→ PAYWALL personnalisé (headline = goal, bullets = pain points, CTA = action verb)
→ Account/email (pour save/sync, jamais avant)
→ Permission au moment d'usage
→ Home avec next action
```

### Étape 4 — Copy concrète

Pour chaque écran, livrer copy précise (voir [08-copy-library.md](references/08-copy-library.md) pour bibliothèque) :

- **Pas de features** → résultats / outcomes
- **Une idée par question, 4-5 options mutuellement exclusives**
- **Sensitive questions** : précéder d'un "On te demande ça pour adapter le plan, pas pour te juger"
- **Always provide escape** : "Skip" / "Je ne sais pas encore" / "Autre"
- **Loading copy honnête** : "On calcule ton plan à partir de tes réponses" (pas "Analyse en cours" générique)
- **Paywall headline = goal personnalisé** : "Débloque ton plan 14 jours [goal]" pas "Choose a plan"

### Étape 5 — Logique de personnalisation

Décrire le mapping `réponses → outputs` :

- Variables collectées (typed) → segment ID
- Segment → plan template / paywall variant / pricing
- Code-pattern réutilisable (voir [03-personalization-engine.md](references/03-personalization-engine.md) pour exemples Next.js+Zustand)

**Test de validité** : si deux utilisateurs avec réponses opposées voient le même résultat, ce n'est pas une vraie personnalisation. À supprimer ou refacto.

### Étape 6 — Paywall design + intégration

Voir [04-paywall-integration.md](references/04-paywall-integration.md). En résumé :

- **Layout** : single-page, no scroll, 3-5 icon-bullets (pas de table sur mobile), horizontal plan list, badge "Most Popular" sur target.
- **Trial mechanic** : iOS = pas de toggle paywall (mort Jan 2026, rejet 3.1.2). Use **Blinkist timeline pattern** (Today → Day X reminder → Day Y charge) → +23% trial conv prouvé, -55% complaints.
- **Pricing framing** : annual preselect, per-week display ("$0.76/week"), "SAVE 75%" badge.
- **Social proof** : user count + 4.2-4.5★ + review count + 1-2 testimonials.
- **CTA** : "Continue" / "Start Free Trial" / "Try for $0.00" + "No commitment, cancel anytime".
- **Close button visible** (Apple HIG, sinon rejet Review).
- **Exit drawer** (reframe → downsell → OTO 25-33%) capable de capter +15-20% revenue, mais cap 2-3 expositions et audit Apple.

### Étape 7 — A/B test backlog priorisé

Ne pas tester au hasard. Prioriser par **win rate Adapty 2026** :

| Test type | Win rate LTV | Priorité |
|---|---|---|
| Localization | **62.3%** | #1 (le plus underestimated) |
| Trial structure | 59.6% | #2 |
| Plan duration | 58.7% | #3 |
| Number of plans | 57.1% | #4 |
| Price changes | 45.5% LTV / 28.3% raw conv | #5 (lift $, pas %) |
| Visual/copy-only | 34.6% | #6 (bottom) |

Les apps qui font **14.7+ expériences/an gagnent jusqu'à 40x revenue** vs non-testers. Voir [09-ab-test-playbook.md](references/09-ab-test-playbook.md).

### Étape 8 — Instrumentation analytics

Évent taxonomy minimum :

```
onboarding_started
onboarding_step_viewed     (step_index, step_label)
onboarding_step_completed
question_answered          (question_id, answer_id, segment)
insight_viewed             (insight_id)
loading_viewed
result_viewed              (segment, score, plan_id)
paywall_viewed             (paywall_variant, segment, trial_duration, plan_interval, price)
paywall_plan_selected
paywall_closed             (reason: x | back | scroll_off)
trial_started
purchase_completed         (price, currency, plan_interval)
exit_offer_viewed
exit_offer_accepted
permission_priming_viewed
permission_granted
signup_completed
first_value_completed
d0_returned, d1_returned, d7_retained
trial_cancelled (Day 0/1/2…)
refund_requested
first_renewal
```

Ne JAMAIS optimiser sur trial start sans regarder paid + refund + first_renewal + D7 retention. Voir [11-metrics-and-tracking.md](references/11-metrics-and-tracking.md).

### Étape 9 — Backend admin "Dashboard Réponse" (sous-skill)

GA4/Mixpanel ne suffisent pas pour une équipe support/produit/sales : il faut **un back-office maison** qui montre, par session, les réponses brutes + l'état conversion + un funnel par étape avec drop-off + la distribution des réponses par option. C'est le sous-skill **Dashboard Réponse** ([12-admin-dashboard.md](references/12-admin-dashboard.md)).

Trigger spécifique : "back-office", "dashboard admin", "voir les réponses", "drop-off rate par étape", "completion rate", "voir une session", "export CSV des leads".

Le sous-skill livre, en tournant sur n'importe quel quiz funnel (schema-agnostic) :

- **Data model Prisma** : `QuizSession` (identité + answers JSON + UTM + état conversion + resultData) + `QuizEvent` (event log granulaire pour le vrai funnel et le temps par étape).
- **Config quiz unique** (`lib/quiz/config.ts`) — source de vérité partagée front/admin pour les labels des questions et options.
- **35+ métriques calculées** : volume (24h/7j/30j), engagement (avg/median step reached, bounce, completion, top exit step), capture email, conversion (rate, time-to-pay, refund), funnel par étape avec drop-off absolu et %, distribution des réponses par option × lift conversion vs taux global, top sources UTM/pays/device, tendance journalière 14j, segments convertisseurs.
- **4 pages admin** : overview KPI/funnel/trend, sessions list filtrable, session detail (toutes réponses + timeline events + raw JSON), distribution réponses par question.
- **Export CSV** sécurisé (`/api/admin/export`).
- **Auth fail-closed** via middleware Basic Auth (ou NextAuth/Clerk) — refuser 503 si secret absent.
- **Checklist d'intégration en 8 étapes** pour plug le sous-skill sur un projet existant.

À utiliser en complément de l'étape 8 (instrumentation events) : les events alimentent le dashboard, le dashboard alimente la décision growth.

### Étape 10 — Output format

Selon la requête, livrer :

- **Audit** → SCORE framework (Specific / Commitment / Output / Reduced friction / Experimentation), section par section, avec issue → impact → reco → priority.
- **Strategy** → funnel map + screen-by-screen + copy + perso logic + paywall sequencing + events + A/B backlog + risks.
- **Code** → composants React/Next.js avec store Zustand typed + persistance + analytics wired + tracking events. Stack de référence : Next.js 14 App Router + TypeScript + Tailwind + Framer Motion + Zustand persist + Prisma/Postgres + Stripe webhook. Voir [10-tech-stack.md](references/10-tech-stack.md).
- **Teardown** → flow map + copy verbatim + tactiques psycho + chiffres connus + ce qui s'applique au cas user.

---

## Tableau de référence rapide — chiffres signature 2026 à connaître

| Chiffre | Source | Use |
|---|---|---|
| **Hard paywall 5x conversion vs freemium** (D35 10.7% vs 2.1%) | RevenueCat 2026 | Argumentaire pour basculer freemium → hard si CAC le permet |
| **Y1 retention identique** hard ~27% vs freemium ~28% | RevenueCat 2026 | Démonter le mythe "freemium retains better" |
| **Apps 14.7+ exp/year = jusqu'à 40x revenue** | Adapty 2026 | Justifier investissement testing |
| **89.4-90% trial starts Day 0** | RevenueCat / Adapty 2026 | Pourquoi Day 0 = tout |
| **84% trial 3j cancellations Day 0-1** | RevenueCat 2026 | 3j est viable, pas besoin de plus dans la plupart cas |
| **Trial 17-32j convertit 1.7x mieux que ≤4j** (42.5% vs 25.5%) | RevenueCat 2026 | Mais seulement si la catégorie supporte (Business/Productivity) |
| **Annual default = 2-3x annual mix, +70% revenue** | Sunflower/Stormy | Toujours preselect annual sauf gaming |
| **"Continue" CTA beat descriptive +111%** | Stormy 4500+ A/B | Default pour CTA paywall |
| **Blinkist Honest Paywall +23% trial conv, -55% complaints** | Growth.design / Purchasely | Pattern timeline = standard 2026 |
| **Localization test win rate 62.3%** (le plus haut) | Adapty 2026 | Prio #1 pour A/B |
| **Web-only loses 6.5% net vs IAP** sur iOS US | RevenueCat A/B test | Hybrid IAP+web, jamais web-only |
| **Toggle paywall mort iOS Jan 2026** (rejet 3.1.2) | RevenueCat | Use Blinkist timeline pattern |
| **Weekly plans = 55.6% revenue mix 2025** (vs 43.3% 2024) | Adapty 2026 | Weekly+3d trial = 1.5x LTV |
| **44.5% des purchases Day 0** | Adapty 2026 | Optimiser activation J0 |
| **Cal AI 75% off annual vs monthly** (vs 50% standard) | Adapty | Anchor agressif marche |
| **Noom 113 écrans, $750M ARR** | Retention.blog | Long quiz peut marcher si chaque écran paie |
| **Reverse trial +10 à +40% freemium→premium** | Elena Verna / Dropbox | Pour SaaS PLG mature (Notion/Canva/Superhuman) |
| **Pricing 14j SaaS sweet spot 62% trials** | ChartMogul | Avec achievement-based upgrade triggers (+258% conv) |

---

## Références détaillées

- **[01-conversion-framework.md](references/01-conversion-framework.md)** — Modèle mental "conversion de croyance", SCORE audit, Day 0, psychologie (commitment, goal-gradient, endowed progress, loss aversion, social proof, curiosity gap)
- **[02-questionnaire-design.md](references/02-questionnaire-design.md)** — Types Q, ordre, copy rules, micro-insights, longueur, sensitive questions, mauvaises questions typiques
- **[03-personalization-engine.md](references/03-personalization-engine.md)** — Mapping réponses → segments → plan/paywall, logic patterns avec exemples code TypeScript
- **[04-paywall-integration.md](references/04-paywall-integration.md)** — Patterns A-H (Quiz→Plan, Honest Timeline, Anchor&Decoy, Feature-Gate, Usage-Limit, Now-or-Never, Social Proof, One-time), trial mechanics, exit drawer, Apple/Google compliance
- **[05-paywall-benchmarks-2026.md](references/05-paywall-benchmarks-2026.md)** — Chiffres RevenueCat (115k apps, $16B) + Adapty (16k apps, $3B) 2026, breakdown par catégorie, géographie, plateforme, prix
- **[06-mobile-teardowns.md](references/06-mobile-teardowns.md)** — Cal AI, Noom, BetterMe, Flo, Opal, Stoic, Blinkist, Strava, Calm, Headspace, MyFitnessPal, Photoroom, Remini, Fitbod, Rizz, Reflectly, Finch, YAZIO, Speak, Lose It!, MacroFactor, Captions, Forest, Freedom, ChatGPT/Claude/Perplexity
- **[07-saas-pricing-pages.md](references/07-saas-pricing-pages.md)** — Linear, Notion, Anthropic, Figma, Vercel, Stripe, Cursor, Superhuman, Raycast — pricing structure, badges, toggle, CTA, comparison table, usage-based, reverse trial mechanics
- **[08-copy-library.md](references/08-copy-library.md)** — Headlines, sub-titles, CTAs, micro-insights, social proof, FAQ, loss aversion (FR + EN), templates par catégorie
- **[09-ab-test-playbook.md](references/09-ab-test-playbook.md)** — Win rates, prio, hypothèses concrètes, design d'expérience, statistical significance pratique, anti-patterns
- **[10-tech-stack.md](references/10-tech-stack.md)** — Stack Next.js + Zustand + Prisma + Stripe / RevenueCat / Superwall / Adapty / Purchasely, architecture quiz, persistance state, audit async, webhook flow
- **[11-metrics-and-tracking.md](references/11-metrics-and-tracking.md)** — Event taxonomy, properties, funnel, anti-vanity-metrics, GA4 + Meta Pixel + Mixpanel + RevenueCat events
- **[12-admin-dashboard.md](references/12-admin-dashboard.md)** — **Sous-skill "Dashboard Réponse"**. Backend admin complet schema-agnostic : data model Prisma (`QuizSession` + `QuizEvent`), config quiz centralisée, `lib/admin/stats.ts` exhaustif (35+ métriques : drop-off, completion, conversion, lift par option, cohortes, top sources, time-to-pay), 4 pages admin (overview / session detail / answers / list), endpoint export CSV, middleware Basic Auth fail-closed, checklist d'intégration en 8 étapes. Inspiré du back-office `audit.lazyrank.io/admin` mais générique pour tout quiz (santé, fitness, SaaS, audit, finance, dating, education).

---

## Anti-patterns à refuser systématiquement

1. **Optimiser trial-start sans regarder paid/refund/renewal** — le trial start c'est de la vanity. La métrique reine est **net revenue per install** ou **D60 LTV**.
2. **Quiz long sans personnalisation visible** — Si après 12 questions le user voit le même paywall générique, le quiz est de la friction.
3. **Toggle paywall sur iOS** (mort Jan 2026, rejet Apple Guideline 3.1.2). Use Blinkist timeline.
4. **Comparison table sur paywall mobile** (lose vs bullets, Stormy 4500+ tests). OK sur SaaS pricing pages.
5. **Email/account avant valeur** (Baymard, 42% sites détournent encore trop tôt).
6. **Permissions enchaînées au lancement** (Apple HIG, Android best practices).
7. **Progress bar mensongère** (95% pendant 20 écrans = trust killer).
8. **Hidden close button** (rejet Apple Review) — sauf hard paywall stricte avec `.storeButton(.hidden, for: .cancellation)`.
9. **Discount sur le main paywall** (Adapty 2026 : entraîne users à attendre, kill perceived value). Use post-close welcome offer pour non-converters seulement.
10. **A/B test sans hypothèse** ou statistical power insuffisant — reférer à [09](references/09-ab-test-playbook.md).
11. **Copier la longueur de Noom** sans copier sa logique — 113 écrans marchent SI chaque écran paie son loyer (engagement progressif, micro-insights, social proof aux étapes 5/11). Sinon c'est juste de la friction.
12. **Hard paywall pour produit social/réseau** (BeReal, Locket — paywaller tue le network effect). Choix par modèle économique, pas idéologie.

---

## Décisions où il faut PUSH BACK quand l'utilisateur insiste sur le mauvais choix

- "On veut un toggle paywall iOS" → "Mort depuis Jan 2026 (rejet App Review). Pattern timeline Blinkist = +23% trial prouvé, équivalent fonctionnel."
- "On veut freemium parce que c'est plus user-friendly" → "RevenueCat 2026 sur 115k apps : Y1 retention identique. Hard paywall 5x conversion. Sauf si modèle réseau/social (BeReal-like), hard paywall est probablement le bon choix."
- "On veut un trial 30 jours pour rassurer" → "84% des cancel arrivent J0-1 sur 3j. Long trial inflate trial-start mais ne lift pas paid significatif sauf Business/Productivity. Trial 3-7j sur annual plan = top config Adapty."
- "On veut afficher 5 plans pour donner le choix" → "Choice paralysis. Stormy/Adapty: single plan visible + 'View all plans' link gagne souvent. Sinon 2-3 max avec badge Most Popular sur target."
- "On veut tester le copy en premier" → "Visual/copy-only A/B = 34.6% win rate (le plus bas). Localization 62.3%, trial structure 59.6%, plan duration 58.7% gagnent plus souvent. Test ces leviers d'abord."
- "On va lancer 8 tests en parallèle" → "Statistical power. Lance 1-2 tests à la fois avec sample size calculé. Top apps font 14.7 tests/an, pas 14.7 tests/semaine."

---

## Output format default

Quand l'utilisateur demande une stratégie complète, livrer dans cet ordre :

1. **TL;DR** (3-5 lignes) — modèle économique reco, paywall placement, trial mechanic, prio test #1.
2. **Funnel map** (ascii ou markdown table, 12-20 écrans).
3. **Screen-by-screen** : objectif, copy, perso, events.
4. **Personalization logic** : map réponses → segment → paywall variant.
5. **Paywall design** : layout, trial, pricing, social proof, CTA.
6. **A/B test backlog** : 5-10 tests priorisés par win-rate Adapty.
7. **Events** : taxonomy minimale.
8. **Risks & red flags** : 3-5 anti-patterns à éviter dans le contexte spécifique.

Pour code → fichier ou snippet directement utilisable, pas de pseudo-code.

Pour audit → tableau Issue / Impact / Reco / Priority (P0/P1/P2).

---

## Tone

- **Direct, anti-bullshit, vouvoiement off** (en français) ou **second person** (en anglais).
- **Pas de hype ni de promesses pseudo-scientifiques** (ex. "neuro-marketing prouve…" → bannir).
- **Cite les chiffres avec source** quand ça compte (RevenueCat 2026 / Adapty 2026 / Stormy / Superwall / Growth.design).
- **Push back sur les choix marketing fragiles** plutôt que valider par défaut.

---
> Source: [gquthier/quiz-funnel-expert](https://github.com/gquthier/quiz-funnel-expert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
