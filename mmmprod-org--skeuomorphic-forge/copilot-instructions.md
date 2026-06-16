## skeuomorphic-forge

> This skill should be used when the user asks to build physically-realistic skeuomorphic UI with Tailwind CSS — buttons, panels, gauges, knobs, CRT/LED displays, glass/metal effects, particle systems, industrial hardware. Triggers on requests mentioning: skeuomorphic, realistic depth, industrial UI, 3D button, gauge, meter, analog, tactile, material texture, retro-industrial, aerospace panel, DSP cockpit, VU meter, rotary knob, CRT display, rim light, metal recess, faceplate. Provides shadow stacks, material textures, lighting rules, and construction blueprints. Do NOT trigger for flat/minimal UI or standard Material/Shadcn components.


# SKEUOMORPHIC FORGE — Instructions de generation UI

Ce skill produit de l'UI skeuomorphique physiquement realiste. Chaque composant genere DOIT ressembler a un objet physique reel (metal usine, verre CRT, aluminium brosse, Bakelite), pas un div plat avec une ombre CSS.

Generer du code React/JSX avec Tailwind CSS + inline styles (`style={{}}`). Pas de classes CSS custom sauf indication contraire.

---

## CONTRAINTES PHYSIQUES ABSOLUES (s'appliquent a CHAQUE composant)

Violer une seule de ces regles produit un composant visuellement casse.

**C1 Ombres portees = NOIR UNIQUEMENT.** Tous les `box-shadow` non-inset utilisent exclusivement `rgba(0,0,0,...)`. Une ombre est l'absence de lumiere. Jamais d'ombre portee violette, ambree, bleue. Exceptions : (a) les `inset` highlights avec rgba clair/colore sont OK (lumiere frappant les bords internes), (b) les glows d'emission/backlight a tres faible opacite (< 0.08) sont OK pour simuler la lumiere emise par un ecran/LED (ex: `0 0 60px rgba(255,176,0,0.06)`).

**C2 Source lumineuse unique a 135deg**, coherente sur TOUS les composants de la page. Plusieurs directions = physiquement impossible.

**C3 Highlights chauds au-dessus de 0.10 d'opacite.** Tout `rgba(255,255,255,X)` avec X > 0.10 doit devenir `rgba(255,240,220,X)` ou plus chaud. Blanc pur au-dessus de 0.10 = blafard/clinique. Les edge catches a 0.03-0.08 peuvent rester en blanc pur.

**C4 Separation de contraste.** Les fonds de conteneurs/wells (#08-#10) DOIVENT etre plus sombres que les fonds de cartes (#1c-#28). Delta minimum #12 hex. Renforcer les bords de carte : `borderTop: "1px solid rgba(255,255,255,0.09)"`, `borderLeft: "1px solid rgba(255,255,255,0.06)"`.

**C5 Ombre et lumiere sont des systemes SEPARES.** L'ombre (rgba sombre) cree profondeur/recession via `box-shadow`. La lumiere (rgba chaud, gradients, pseudo-elements) cree reflets speculaires/edge catches. Ne jamais les confondre dans un stack brouille.

**C6 Minimum de couches d'ombre.** Standard (boutons/cartes/toggles) >= 5. Advanced (knobs/dials/meters) >= 8. Hero (panneaux/chassis) >= 11. Ultra (CRT vitrine) = 31. LES COMPTER apres collage.

**C7 Ne jamais inventer de shadow stacks.** Toujours copier-adapter depuis les stacks canoniques ci-dessous.

**C8 Ordre d'assemblage : arriere vers avant.** Backplate -> sous-panneaux/bezels -> wells/recesses -> instruments/displays -> hardware (vis/rivets) -> labels -> verre/reflet. Du hardware sur une surface plate = contradiction. Construire la profondeur D'ABORD.

**C9 Taille physique fixe.** Les cartes-appareils (CRT, gauge, instrument) ont des dimensions FIXES. Toutes les instances du meme composant = meme width+height. Etat disabled = memes dimensions que actif. Le contenu se centre dans le chassis. Utiliser `width` + `height` explicites.

**C10 Max 2 couleurs d'accent par page.** Un 3e accent necessite approbation explicite.

---

## VERIFICATION PRE-LIVRAISON (checker AVANT de livrer)

Un composant qui echoue un gate CRITICAL ne doit PAS etre livre.

**U0 Context Scan (BLOCKING):** La page existante a ete analysee avant de styler. Palette, boutons existants, hierarchie des conteneurs identifies. Meme role = meme style que les siblings. Shadow stack source depuis les canoniques, pas invente.

**U1 Shadow Depth (CRITICAL):** Nombre de couches respecte le minimum du tier (C6). Ombres portees NOIR uniquement (C1). Progression graduee du blur.

**U2 Light Source (CRITICAL):** Direction unique 135deg partout (C2). Lumiere/ombre separees (C5). Pas de blanc pur au-dessus de 0.10 (C3). Gradient de surface present.

**U3 Construction (HIGH):** Objet physique nomme explicitement. 4 couches : chassis + profondeur + eclairage + detail. Ordre d'assemblage respecte (C8).

**U4 Hardware (HIGH):** Vis aux coins avec radial-gradient sphere + 5 couches d'ombre + slot torx/phillips. Vis sur METAL uniquement, jamais sur verre/ecran.

**U5 Interaction States (HIGH):** hover (lift + expand shadow), active (depress + compress), disabled (opacity 0.5 + desaturate), focus-visible (outline 2px). Le shadow stack CHANGE entre les etats.

**U6 Typography (HIGH):** Body >= 13px, titres >= 14px, labels >= 11px, rien sous 10px. Texte primaire opacity >= 0.85, secondaire >= 0.5, tertiaire >= 0.35. Labels en silkscreen (text-shadow) ou grave (clip + gradient), pas en texte brut.

**U7 Accessibility (MEDIUM):** Contraste WCAG OK. `focus-visible` sur tous les interactifs. Touch targets >= 44px. `prefers-reduced-motion`. `pointer-events: none` sur les overlays de texture. Pas de `filter: blur()` dans les animations.

---

## SHADOW STACKS CANONIQUES (copier-adapter, ne jamais inventer)

Les quatre tiers de shadow stacks — **Standard** (5+ couches), **Advanced** (8-9), **Hero** (11+ minimum, golden example en utilise 13+), **Ultra** (31) — sont documentes avec code verbatim dans `references/00-golden-examples.md` section `## 1. SHADOW STACKS BY TIER`. **Charger ce fichier AVANT de construire tout composant avec relief.** Ne jamais reconstruire un stack de memoire : copier-adapter depuis le golden file.

Rappel des tiers (pour classifier rapidement, voir tableau CLASSIFICATION DU COMPOSANT ci-dessous) :
- **Standard 5+** : boutons, cartes, toggles
- **Advanced 8-9** : knobs, dials, meters, rails de switch, display wells
- **Hero 11+ minimum (golden = 13+)** : panneaux, chassis, faceplates
- **Ultra 31** : deep CRT recess

---

## CLASSIFICATION DU COMPOSANT

### Boutons

| Role | Tier shadow | Gradient | Taille |
|---|---|---|---|
| CTA/Hero | Advanced 8+ | Bold thematique + shimmer/LED | 48-56px height, 1 par page max |
| Primary | Standard 5+ | Surface gradient accent | 40-44px |
| Secondary | Standard 5 | Subtle surface gradient | 36-40px |
| Tertiary/Ghost | Minimal 2-3 inset (exception C6 — ghost = pas de relief physique) | Aucun, transparent + border | 32-40px |
| Destructive | Standard 5+ | Red `#4a1010->#2a0808` | 40-44px |

### Conteneurs

| Role | Background | Shadow |
|---|---|---|
| Section well | #08-#0e (le plus sombre) | Inset 4-6 couches |
| Card (flottante) | #1c-#28 (plus clair) | Drop Standard 5+ |
| Card imbriquee | #14-#1c (intermediaire) | Drop 3-5 |
| Modal/overlay | #1a-#22 | Hero 11+ |

### Elements speciaux

| Type | Tier shadow |
|---|---|
| Gauge/meter | Advanced 8+ |
| CRT display | Hero 11+ ou Ultra 31 |
| Toggle/switch | Standard 5 |
| Knob/dial | Advanced 8+ |
| LED indicator | Standard 5 |

---

## PROTOCOLE DE CONSTRUCTION

### 1. Nommer l'objet physique

Dire explicitement : "Ceci est une faceplate en aluminium usine" ou "Ceci est un interrupteur a bascule en Bakelite". Si l'analogie physique est floue, le composant sera generique.

### 2. Choisir la famille esthetique

| Theme | Surface gradient | Accent | Inset highlight | Texte |
|---|---|---|---|---|
| Warm industrial (defaut) | `#1e1e1e->#141414` | `hsl(35 100% 60%)` amber | `rgba(255,240,220,0.12)` | `rgba(255,240,220,0.9)` |
| Cool steel | `#1a1c20->#12141a` | `hsl(210 70% 60%)` blue | `rgba(180,200,255,0.10)` | `rgba(200,220,255,0.9)` |
| Deep purple | `#2d1854->#150b28` | `hsl(270 100% 70%)` violet | `rgba(200,160,255,0.12)` | `rgba(220,200,255,0.95)` |
| Military | `#1a1e14->#10140c` | `hsl(0 70% 55%)` red | `rgba(180,200,150,0.08)` | `rgba(180,200,150,0.85)` |
| Vintage audio | `#2a2218->#1a1610` | `hsl(30 80% 55%)` copper | `rgba(255,220,180,0.10)` | `rgba(240,220,190,0.9)` |
| CRT terminal | `#0a0f0a->#050805` | `hsl(120 100% 50%)` green | `rgba(0,255,60,0.06)` | `rgba(0,255,60,0.85)` |

Les ombres portees restent NOIRES `rgba(0,0,0,...)` quel que soit le theme.

### 3. Eclairage (applique APRES les ombres)

1. **Gradient de surface** (courbure) : `background: linear-gradient(145deg, plus-clair 0%, plus-sombre 100%)`
2. **Bords biseautes** : `borderTop: "1px solid rgba(255,255,255,0.08)"`, `borderBottom: "1px solid rgba(0,0,0,0.3)"`
3. **Point speculaire** : `::before` avec `radial-gradient(circle at 35% 30%, rgba(255,240,220,0.2), transparent 60%)`
4. **Rim glow** : `::before` avec `radial-gradient(ellipse at center, rgba(255,255,255,0.25), transparent 70%)` au bord superieur, `pointerEvents: "none"`

| But | Couleur | Opacite |
|---|---|---|
| Edge catch | `rgba(255,255,255,...)` | 0.03-0.08 |
| Top bevel | `rgba(255,255,255,...)` | 0.08-0.15 |
| Specular hotspot | `rgba(255,240,220,...)` | 0.15-0.30 |
| Active glow | `rgba(255,180,60,...)` | 0.10-0.40 |
| CRT/LED emission | `hsl(35 100% 60%)` | Full |
| Rim light (pseudo) | `rgba(255,255,255,0.25)` center->transparent | radial-gradient |

### 4. Etats d'interaction (CHAQUE element interactif)

```javascript
// Hover — lift + expand shadow
onHover: { transform: 'translateY(-1px)', boxShadow: '/* meme stack, blur +2px/couche, highlight opacity +0.05 */' }
// Active — depress
onActive: { transform: 'translateY(1px)', boxShadow: 'inset 0 2px 4px rgba(0,0,0,0.5), inset 0 1px 1px rgba(0,0,0,0.3), 0 1px 2px rgba(0,0,0,0.3)' }
// Disabled
onDisabled: { opacity: 0.5, filter: 'saturate(0.3)', pointerEvents: 'none' }
// Focus
focusVisible: { outline: '2px solid hsl(35 100% 60%)', outlineOffset: '2px' }
```

---

## REGLES COMPOSANTS SPECIFIQUES

### Metal Recesses / Wells

Chaque recess necessite les 4 zones : top attack, murs lateraux, bottom catch, levre exterieure.

| Composant | Min couches inset | Background |
|---|---|---|
| Input / petit statut | 6 | `#080808->#0c0c0c` |
| Well de gauge/meter | 9 | `#050505->#080808` |
| CRT/deep display | 12 | `#030303` |
| Canal slider | 5 | gradient `#050505->#0f0f0f` |
| Trou LED | 4 | radial `#1a1a1a->#000` |

**INTERDIT** : `inset 0 2px 4px rgba(0,0,0,0.3)` seul (1-3 couches). C'est un input CSS, pas un recess metal.

### Rim Light

Shadow stack construit D'ABORD (5+ couches), PUIS rim light par-dessus. Bord superieur 2-4x plus lumineux que le bas. Au moins UN pseudo-element overlay (`<span>` absolu ou Tailwind `before:` / `after:`) avec `radial-gradient` pour hotspot concentre. Rim light opacity max 0.25 (specular hotspots et active glows peuvent aller plus haut — voir tableau Eclairage). `pointerEvents: "none"` sur les overlays.

**INTERDIT** : `border: 1px solid rgba(255,255,255,0.1)` comme "rim light" (border uniforme = cadre photo). `box-shadow: 0 0 20px rgba(255,255,255,0.1)` seul (glow uniforme = neon).

**Note pseudo-elements** : En JSX inline, `::before`/`::after` ne sont pas disponibles. Utiliser soit des classes Tailwind (`before:content-[''] before:absolute before:...`), soit un `<span aria-hidden="true" className="pointer-events-none absolute ..." />` comme overlay dedié.

### Chassis Metal / Panneaux

3+ zones obligatoires parmi : (1) Cadre bezel (rim multi-couches + bevel borders), (2) Surface principale (metal brosse/texture, PAS gris plat), (3) Wells/recesses, (4) Zones d'affichage (recessed + color bleed), (5) Grille/ventilation (hex perf ou fentes), (6) Labels (estampes/graves, PAS texte brut).

**INTERDIT** : Rectangle plat `#333` avec grille CSS de points. Tout chassis a besoin de texture de surface, au moins un recess, au moins un detail hardware.

---

## PATTERNS DE PRODUCTION CANONIQUES

Tous les patterns prets-a-l'emploi (bouton rest/hover/active, card avec rim light, tete de vis, phosphor/CRT glow, silkscreen label, gradients de materiaux metal/chrome/cuir/bois) sont documentes avec code verbatim dans `references/00-golden-examples.md`. Les references utilisent les titres exacts (`## N. NAME`) du golden file pour resister aux renumerotations :

| Pattern | Section dans `references/00-golden-examples.md` |
|---|---|
| Bouton complet Rest/Hover/Active | `## 2. COMPLETE BUTTON — Rest / Hover / Active` |
| Industrial Circuit Relay Button | `### Industrial Circuit Relay Button` |
| Card avec Rim Light | `## 3. COMPLETE CARD WITH RIM LIGHT` |
| Tete de vis (5 couches + slot torx) | `## 4. SCREW HEAD (5 layers + radial gradient)` |
| Phosphor / CRT Text Glow (amber, rouge) | `## 5. PHOSPHOR / CRT TEXT GLOW` |
| Silkscreen Label | `## 6. SILKSCREEN LABEL` |
| Metal brosse / Sphere knob / Chrome / Cuir / Bois | `## 7. MATERIAL SURFACE GRADIENTS` |

**Workflow** : charger `references/00-golden-examples.md`, trouver le pattern par son titre exact, copier le bloc, adapter les couleurs/tailles au theme cible. **IMPORTANT** : les blocs golden sont ecrits en CSS pur (`.button { box-shadow: ... }`). Conformement a la regle de generation (React/JSX + Tailwind + inline styles), **traduire chaque bloc CSS en `style={{ boxShadow: '...' }}` ou en classes Tailwind avant utilisation** — ne jamais coller le CSS verbatim dans une feuille de style globale ni creer de classes custom. Pour des materiaux avances (aluminium brosse detaille, acier, cuivre) consulter `references/08-metal-effects.md`. Pour le verre consulter `references/07-glass-effects.md`.

Une etape CI (`Validate golden-examples references` dans `skill-integrity.yml`) verifie qu'a chaque release, tous les titres de section listes ci-dessus existent encore dans `references/00-golden-examples.md`. Si la verification echoue, soit le golden file a ete renomme, soit ce tableau doit etre mis a jour.

---

## PALETTE & TYPOGRAPHIE

La palette **WARM INDUSTRIAL** (defaut — 13 tokens : chassis-dark/base/mid/light, bevel, highlight, amber/amber-dim/amber-glow, cream, green, red, silk) et la table **Typographie** (CRT/VFD Orbitron, Terminal Share Tech Mono, Counter digits, Silkscreen, Gravure) sont documentees dans `references/00-golden-examples.md` sections `## 8. WARM INDUSTRIAL PALETTE` et `## 9. TYPOGRAPHY`. La version reference inclut `--ri-silk-active` (label backlit) en bonus.

Pour les autres themes (cool steel, deep purple, military, vintage audio, CRT terminal) voir la section `### 2. Choisir la famille esthetique` sous `## PROTOCOLE DE CONSTRUCTION` ci-dessus. Pour les palettes etendues et tokens de patina/vieillissement voir `references/06-aging-safety-tokens-palettes.md`.

---

## VOCABULAIRE 3D

| L'utilisateur dit | Technique |
|---|---|
| "3D effect" / "effet 3D" | Multi-layer shadow stack + perspective tilt optionnel |
| "3D button" | Press effect : compression shadow sur :active |
| "depth" / "profondeur" | Couches d'ombre graduees OU translateZ |
| "relief" / "embossed" | Texte/forme sureleve via gradient clip + text-shadow |
| "recessed" / "inset" | Inset shadow stack (effet gorge) |
| "floating" / "levitating" | Drop shadows extra-larges + translateY lift |
| "parallax" | Couches a differents translateZ ou vitesses de scroll |
| "glass dome" / "bubble" | Pseudo-element radial-gradient sur le contenu |
| "flip card" | rotateY(180deg) + preserve-3d + backface-visibility |
| "isometric" | rotateX + rotateY petits angles + faces d'arete |

---

## ANTI-PATTERNS

| MAUVAIS | BON |
|---|---|
| 2-3 couches box-shadow | 5-15 couches, blur gradue |
| Vis sur surface plate | Profondeur D'ABORD, vis par-dessus |
| Vis sur verre/ecran | Vis sur METAL uniquement |
| Vis cercle plat 1 inset | Radial gradient sphere + 5 couches + slot torx |
| Carte qui resize avec contenu | width+height explicites, frame fixe |
| Recess sans inner rim light | 1px warm border au bord, top plus lumineux |
| 1-3 inset pour ecran | 12+ couches inset + inner rim light |
| Rim light avec glow/blur | Border clean 1px, PAS de bloom |
| Pas de color bleed depuis display | Inner rim capte la couleur display a 0.06-0.10 |
| Texte embosse opacity < 0.25 | Min 0.35 tertiaire, 0.50 secondaire |
| Taille differente pour etat disabled | Memes width+height pour tous les etats |
| Rim light decoratif sans source physique | Chaque effet lumineux a une source physique |
| Blanc pur rgba(255,255,255,X) X>0.10 | rgba(255,240,220,X) tint chaud |
| Meme box-shadow rest/hover/active | Stacks differents par etat |
| `filter: blur()` en animation | opacity + transform uniquement |
| Plusieurs directions lumineuses | 135deg unique partout |
| Inventer des shadow stacks | Copier-adapter depuis les canoniques |
| Hardware avant profondeur | Ordre d'assemblage : profondeur d'abord |
| Lumiere + ombre melangees | Systemes separes |
| Edge catch rgba(255,255,255,0.5) | Max 0.08 pour les edges |
| Ombres portees colorees | rgba(0,0,0,...) pour TOUTES les drops |
| Body text < 13px | Body >= 13px, titres >= 14px, labels >= 11px |
| Conteneur/carte meme luminosite | Delta minimum #12 hex |
| Toujours amber-on-dark | Proposer 2-3 themes si nouvelle UI |
| Styler sans lire la page | Scanner la page, extraire palette, matcher hierarchie |
| Choisir style bouton sans classifier | Utiliser la matrice de decision |
| Ajouter un 3e accent silencieusement | Max 2 par page, demander avant le 3e |
| Ignorer les siblings | Meme role = meme style |

---

## VARIETE CREATIVE (nouvelles UIs uniquement)

Si modification d'une page existante : coherence, matcher le theme existant. Ne pas proposer de variete.

Si creation d'un composant net-new sans theme etabli : proposer 2 options maximum depuis les 6 themes ci-dessus. L'utilisateur choisit.

Ne jamais proposer de variete quand la page environnante etablit deja le role, la palette et la hierarchie.

---

## HTML ASSETS — Lookup rapide

| Besoin | Fichier | Cle recherche |
|---|---|---|
| Push button avec LED | `power-button.html` | `power-button` |
| Dashboard / analytics | `synthscore-analytics.html` | `synthscore` |
| VU meter / gauge | `tube-compressor-vu.html` | `vu-meter` |
| Toggle switch | `skeuomorphic-switch.html` | `switch` |
| LCD numeric display | `lcd-db-display.html` | `lcd` |
| Rotary lever / knob | `industrial-lever.html` | `lever` |
| Score gauge / radial | `credit-score-gauge.html` | `gauge` |
| Alert / warning panel | `autochord-alert-panel.html` | `alert` |
| CRT danger screen | `deep-screen-phosphor.html` | `phosphor` |
| Folding / accordion | `folding-header.html` | `folding` |
| Horizontal bar meter | `horizontal-thermometer.html` | `thermometer` |
| Deep CRT (31 layers) | `codepen-deep-screen.html` | `deep-screen` |
| Full page (15+ buttons) | `agile-tech-skeuomorphic-site.html` | `site` |
| Rimlight toggle switch | `rimlight-toggle-switch.html` | `rimlight` |
| Color-mix buttons | `color-mix-buttons.html` | `color-mix` |
| Rocker 3D switch | `rocker-3d-switch.html` | `rocker` |
| Tile up/down/button | `tile-buttons-divs.html` | `tiles` |
| Neumorphic loader | `neumorphic-loading-circle.html` | `loading` |
| Progress loader (light) | `neumorphic-progress-loader.html` | `progress` |
| Pressed buttons (light) | `neumorphic-pressed-buttons.html` | `pressed` |
| Fingerprint SVG button | `fingerprint-button.html` | `fingerprint` |

Tous les fichiers dans `assets/` (au niveau root du plugin).

---

## RESSOURCES SUPPLEMENTAIRES

Ce skill expose trois dossiers au niveau root du plugin (`assets/`, `references/`, `scripts/`). Les references contiennent ~108k mots de deep-dive pour sujets qui depassent le scope du SKILL.md principal.

### Reference Files (`references/`)

Charger a la demande selon le sujet de travail :

| Fichier | Contenu | Quand consulter |
|---|---|---|
| `00-golden-examples.md` | Exemples canoniques a copier-adapter | Tout nouveau composant — partir d'un golden exemple |
| `01-shadows-materials-textures.md` | Shadow stacks detailles par materiau + textures SVG | Construire un nouveau tier de shadow stack |
| `02-hardware-animation-neumorphism.md` | Vis, rivets, hinges + animations + neumorphism variants | Ajouter du hardware ou des etats animes |
| `03-blueprints-performance-palettes.md` | Blueprints complets + perf tips + palettes etendues | Composant complexe multi-couches |
| `04-community-techniques.md` | Techniques issues de la communaute (CodePen, Dribbble) | Inspiration pour effets avances |
| `05-physics-composition-interaction-typography.md` | Regles physiques + composition + states + typo detaillee | Debug d'un composant qui parait "faux" |
| `06-aging-safety-tokens-palettes.md` | Patina/vieillissement, design tokens, palettes etendues | Theming global ou finition vintage |
| `07-glass-effects.md` | Verre CRT, domes, lenses, reflets speculaires | Tout ce qui doit ressembler a du verre |
| `08-metal-effects.md` | Aluminium brosse, acier, chrome, cuivre | Surfaces metal texturees |
| `09-rim-light-effects.md` | Rim light avance (beyond le pseudo-element basique) | Quand rim light basique insuffisant |
| `10-particle-effects.md` | Particules, sparks, dust, ambient motion | Fonds animes, feedback visuel |
| `11-retro-industrial-patterns.md` | Patterns complets aerospace / DSP / military cockpit | Theme retro-industrial full-page |
| `12-production-components.md` | Composants production-ready pret a l'emploi | Reutilisation de composants eprouves |
| `13-3d-depth-techniques.md` | Perspective, tilt, parallax, isometric | Effets 3D au-dela des shadow stacks |
| `14-metal-recess-wells.md` | Construction detaillee de wells metalliques | Recess complexe (gauge/meter/display) |
| `15-detailed-chassis.md` | Chassis multi-zones (bezel + surface + wells + grille + labels) | Panneau complet faceplate |
| `16-benchmark-lessons.md` | Lecons des benchmarks (ce qui marche / ce qui casse) | Eviter les pieges connus |
| `17-context-scan-matrices.md` | Matrices U0 Context Scan approfondies | Styler dans une page existante complexe |
| `18-verification-checklist.md` | Checklist verification U0-U7 detaillee avec exemples | Gate final avant livraison |

### Scripts (`scripts/`)

- **`search.py`** — Recherche semantique sur `assets/` et `references/`. Utile pour trouver le bon exemple ou la bonne reference sans charger 108k mots dans le contexte. Executer via `python3 scripts/search.py "query"` (le script utilise des features Python 3 — typing `list[dict]`, etc.).

### Assets (`assets/`)

21 fichiers HTML complets de composants skeuomorphiques production-ready (voir tableau "HTML ASSETS — Lookup rapide" ci-dessus). Copier-adapter plutot que re-inventer.

---
> Source: [MMMProd-Org/skeuomorphic-forge](https://github.com/MMMProd-Org/skeuomorphic-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
