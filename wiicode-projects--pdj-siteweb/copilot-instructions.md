## pdj-siteweb

> Fichier de référence automatiquement chargé par Claude Code à chaque nouvelle session.

# CLAUDE.md — Plat du Jour

Fichier de référence automatiquement chargé par Claude Code à chaque nouvelle session.
Contient tout ce qu'il faut savoir pour travailler sur ce projet sans perdre de contexte.

---

## Identité du projet

**Plat du Jour** est une landing page pour une application mobile suisse de découverte des menus du jour.
L'app permet aux utilisateurs de trouver les plats du jour autour d'eux (géolocalisation), de réserver,
et aux restaurateurs de publier leur menu facilement.

- **URL de production :** https://plat-du-jour.vercel.app
- **Dépôt GitHub :** https://github.com/magi22/Plat-du-jour
- **Déployé sur :** Vercel (pas Netlify — le netlify.toml a été supprimé)
- **Contact chatbot :** WhatsApp/Telegram +221785421733

---

## Stack technique

| Outil | Version | Rôle |
|---|---|---|
| React | 19 | Framework UI |
| TypeScript | — | Typage |
| Vite | — | Build tool |
| Tailwind CSS | — | Styles utilitaires |
| Framer Motion | 12 | Animations entrées/sorties/présence |
| Radix UI Accordion | 1.2 | FAQ accessible |
| Embla Carousel | 8 | Carrousel témoignages |
| Lucide React | — | Icônes |

---

## Commandes utiles

```bash
npm run dev      # Serveur local
npm run build    # Build production (toujours vérifier avant push)
```

---

## Architecture des fichiers

```
src/
├── App.tsx                          # Séquençage sections + fond animé global
├── index.css                        # Animations custom + fonds animés (classes fr)
├── main.tsx
│
├── i18n/
│   ├── translations.ts              # TOUTES les traductions FR/EN/DE/IT
│   └── LanguageContext.tsx          # Context + useLanguage() hook
│
├── animations/
│   └── variants.ts                  # fadeUp, fadeLeft, fadeRight, scaleIn, staggerContainer, defaultViewport
│
├── hooks/
│   └── useScrolled.ts               # Boolean basé sur scroll threshold
│
├── lib/
│   └── getStoreLink.ts              # Détection iOS/Android → lien store adapté
│
├── constants/
│   └── testimonials.ts              # STATS + TESTIMONIALS (données statiques)
│
├── assets/
│   ├── Mascotte@2x.png              # Mascotte principale (watermark + RestaurantSection)
│   ├── Logomascotte.svg             # Logo (Header, Footer, Hero badge)
│   ├── restaurant (1).png           # Photo RestaurantSection (locale)
│   ├── *shots_so.png (8 fichiers)   # Captures AppShowcase
│   ├── apple3.gif / apple4.png ...  # Icônes Apple Store
│   └── play3.gif / play4.png ...    # Icônes Google Play
│
├── components/
│   ├── Header.tsx
│   ├── Hero.tsx
│   ├── ProblemSolution.tsx
│   ├── HowItWorks.tsx
│   ├── Features.tsx
│   ├── AppShowcase.tsx
│   ├── RestaurantSection.tsx
│   ├── Pricing.tsx
│   ├── Testimonials.tsx
│   ├── Faq.tsx
│   ├── DownloadSection.tsx
│   ├── Footer.tsx
│   │
│   └── ui/
│       ├── LanguageSelector.tsx     # Sélecteur langue (pills horizontaux)
│       ├── ChatBot.tsx              # Widget chat flottant
│       ├── ScrollToTop.tsx          # Bouton remonter en haut
│       ├── FadeIn.tsx               # Wrapper animation réutilisable
│       ├── SectionContainer.tsx     # Wrapper max-w + padding
│       └── ImageWithFallback.tsx    # Image avec fallback
│
public/
└── favicon.svg                      # Copie de Logomascotte.svg
```

---

## Ordre des sections (App.tsx)

```
Header (fixe)
Hero                     → fond clair animé
ProblemSolution          → fond clair animé
── Fondu blanc → gris ──
HowItWorks               → bg-gray-50
── Fondu gris → blanc ──
Features                 → fond clair animé
── Ligne 1px ───────────
AppShowcase              → bg #0a0a0f (noir)
── Ligne 1px ───────────
RestaurantSection        → fond clair animé
── Fondu blanc → gris ──
Pricing                  → bg-gray-50
── Fondu gris → blanc ──
Testimonials             → fond clair animé
Faq                      → fond clair animé
── Ligne 1px ───────────
DownloadSection          → bg #0a0a0f (noir)
Footer                   → bg #070710
[overlay] ChatBot + ScrollToTop
```

**Règle de transition (ne pas modifier) :**
- `blanc ↔ noir` : `<Ligne />` (1px, pas de dégradé)
- `blanc ↔ gris` : `<Fondu />` (40px gradient)

---

## Système i18n

**4 langues :** FR · EN · DE · IT

- Toutes les chaînes sont dans `src/i18n/translations.ts`
- Hook : `const { t, lang, setLang } = useLanguage()`
- Auto-détection : `navigator.language`
- Persistance : `localStorage`
- **Ne jamais hardcoder du texte dans les composants** — tout passe par `t.section.clé`

**Exceptions (données statiques non traduites) :**
- `src/constants/testimonials.ts` : STATS + TESTIMONIALS
- `src/components/RestaurantSection.tsx` : badge `+ 1'250 CHF`
- `src/components/DownloadSection.tsx` : noms des restaurants sur la carte

**Sélecteur de langue :**
- Design : pills horizontaux inline au clic (pas de dropdown vertical — ça se coupe sur mobile)
- Fichier : `src/components/ui/LanguageSelector.tsx`

---

## Système de fond animé (index.css)

Classes CSS en **français** pour lisibilité :

### Fond clair `.fa*` (sections blanches)
```html
<div class="fa">
  <div class="fa-tinte" />   <!-- dégradés radiaux rouge + jaune, animation 12s -->
  <div class="fa-grille" />  <!-- grille 48px qui défile en diagonale, 18s -->
  <div class="fa-lueur fa-l1" />  <!-- cercle rouge haut-gauche, 10s -->
  <div class="fa-lueur fa-l2" />  <!-- cercle jaune bas-droite, delay -3s -->
</div>
```
Placé une seule fois dans App.tsx. Les sections `bg-transparent` laissent transparaître le fond.

### Fond sombre `.fad*` (sections noires)
Même structure mais avec `.fad`, `.fad-tinte`, `.fad-grille`, `.fad-lueur`, `.fad-l1`, `.fad-l2`.
Utilisé dans `DownloadSection.tsx`.

### Animations disponibles
| Classe | Effet |
|---|---|
| `animate-float` | Flottement vertical 10px, 6s |
| `animate-float-delayed` | Flottement 14px, 7s |
| `animate-slide-up` | Apparition par le bas, 0.7s |
| `animate-pop-in` | Scale 0.96→1, 0.5s |
| `animate-secousse` | Secousse rotation ±14°, toutes les 5s (ChatBot) |

---

## Contenu suisse (important)

Le site cible la **Suisse** :
- **Monnaie :** CHF (pas €)
  - User Premium : 4.90 CHF/mois · 49.- CHF/an
  - Restaurant Premium : 49 CHF/mois · 490 CHF/an
  - Badge revenus : `+ 1'250 CHF` (apostrophe = séparateur suisse)
- **Villes** dans les témoignages : Genève, Lausanne, Zurich, Berne
- **Noms** dans les témoignages : Müller, Favre, Rochat, Gerber
- **Restaurants démo** (DownloadSection) : Kornhauskeller, Altes Tramdepot, Rosengarten

---

## Composants UI overlay (ChatBot + ScrollToTop)

### ChatBot (`src/components/ui/ChatBot.tsx`)
- Position : `fixed bottom-[6.5rem] right-6 z-50`
- Bouton : `w-12 h-12 bg-primary border-2 border-white/80 animate-secousse`
- **Bulle d'aide auto :** affichée à 15s, masquée à 45s (30s d'affichage)
- **6 questions :** 3 simples (réponse directe) + 3 complexes (→ WA/TG)
- Redirection : `https://wa.me/221785421733` et `https://t.me/+221785421733`
- Questions complexes : style rouge (`border-red-200 bg-red-50`)
- Fenêtre : `w-[calc(100vw-2rem)] sm:w-80`, `bottom-[10.5rem]`

### ScrollToTop (`src/components/ui/ScrollToTop.tsx`)
- Visible après 400px de scroll
- Position : `fixed bottom-6 right-6 z-50`

---

## Règles de développement

### À toujours respecter
1. **Build avant push** : `npm run build` doit passer sans erreur
2. **Commits en français** avec message concis
3. **Pas de commentaires inutiles** dans le code
4. **Pas de texte hardcodé** — tout dans `translations.ts`
5. **Classes CSS en français** pour les nouvelles classes custom dans `index.css`
6. **Edit ciblé** — préférer `Edit` sur un fichier existant plutôt que `Write` complet

### Bugs connus (déjà corrigés, ne pas réintroduire)
- `key={texteTraduire}` sur `motion.div` avec `viewport={{ once: true }}` → éléments invisibles après changement de langue. **Fix : toujours `key={i}` (index)**
- `FaqColumn` défini à l'intérieur de `Faq` → remontage à chaque render. **Fix : défini à l'extérieur**
- `-mt-2` dans `Accordion.Content` + `overflow-hidden` → texte coupé en haut. **Fix : `pt-2` à la place**
- Dropdown vertical LanguageSelector dans menu mobile → coupé par `overflow-hidden`. **Fix : pills horizontaux**

### Framer Motion
- Toujours utiliser les variants depuis `src/animations/variants.ts`
- `defaultViewport = { once: true }` — les animations ne se rejouent pas
- `staggerContainer` pour les listes d'éléments

---

## Design tokens

| Token | Valeur |
|---|---|
| Couleur primaire | `#c1111e` (rouge vif) |
| Rouge secondaire | `#ff4757` |
| Fond blanc sections | `transparent` (fond animé en dessous) |
| Fond gris sections | `#f9fafb` (`bg-gray-50`) |
| Fond noir sections | `#0a0a0f` |
| Fond footer | `#070710` |
| Police | `font-sans` (système) |

---

## Points d'ancrage (navigation)

| Section | ID |
|---|---|
| Comment ça marche | `#how-it-works` |
| Fonctionnalités | `#features` |
| Restaurants | `#restaurants` |
| Tarifs | `#tarifs` |
| Témoignages | `#testimonials` |
| FAQ | `#faq` |
| Téléchargement | `#download` |

---

## Préférences utilisateur

- **Langue de communication :** Français
- **Pas de commentaires** superflus dans le code
- **Commits en français** — refus des messages en anglais
- **Ne pas recréer** les assets existants — vérifier d'abord dans `src/assets/`
- **Pas de dégradé** entre sections noir↔blanc — ligne simple 1px uniquement
- **Push automatique** après chaque tâche terminée (sauf si demande explicite de vérification)

---
> Source: [wiicode-projects/pdj-siteweb](https://github.com/wiicode-projects/pdj-siteweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
