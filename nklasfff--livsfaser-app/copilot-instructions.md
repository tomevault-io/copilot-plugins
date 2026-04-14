## livsfaser-app

> Progressive Web App (PWA) companion til den danske bog "De 9 Livsfasers Energi" af Isabelle Evita Søndergaard.

# DE 9 LIVSFASERS ENERGI - PWA

## PROJECT OVERVIEW

Progressive Web App (PWA) companion til den danske bog "De 9 Livsfasers Energi" af Isabelle Evita Søndergaard.

**Technology:** Pure HTML5, CSS3, JavaScript - NO build process
**Deployment:** GitHub Pages (static files)
**Language:** Danish (da-DK)
**Target Users:** Danish women aged 28-63 (men tilgængelig for alle kvinder)

**Core Purpose:**
Dynamisk companion til bogen - hjælper brugere med at forstå hvor de er i ALLE fem cyklusser samtidigt (livsfase, årstid, menstruationscyklus, ugedag, døgntime) og hvordan deres fase interagerer med andre (mor, datter, partner).

**Unique Differentiators:**
1. Viser præcis placering i ALLE fem samtidige cyklusser
2. Generationelt matching (visualiserer hvordan forskellige faser mødes)
3. Personlig visdom gennem tracking og refleksion
4. Forstår MÆNDS 8-års cyklus vs KVINDERS 7-års cyklus (kritisk for parforhold)

---

## TECHNICAL STACK

### Core Technologies
```
HTML5 - Semantic markup, no frameworks
CSS3 - CSS Variables, Grid, Flexbox, no preprocessor
JavaScript ES6+ - Vanilla JS, no libraries except where noted
LocalStorage - All data stored locally
Service Worker - Offline functionality
Web App Manifest - PWA installation
```

### File Structure
```
/
├── index.html              # Main app entry
├── manifest.json           # PWA manifest
├── sw.js                   # Service worker
├── css/
│   ├── variables.css       # Colors, spacing, typography
│   ├── base.css           # Reset, base styles
│   ├── components.css     # Reusable components
│   └── screens.css        # Screen-specific styles
├── js/
│   ├── app.js             # Main app logic
│   ├── cycles.js          # Cycle calculations
│   ├── storage.js         # LocalStorage wrapper
│   ├── navigation.js      # Bottom nav & routing
│   └── utils.js           # Helper functions
├── assets/
│   ├── icons/             # App icons (192x192, 512x512)
│   └── images/            # Content images
└── screens/
    ├── idag.html          # I dag screen
    ├── udforskning.html   # Udforskning screen
    ├── relationer.html    # Relationer screen
    └── favoritter.html    # Favoritter screen
```

---

## DESIGN SYSTEM

### Color Palette
```css
/* Primary Colors - EXACT VALUES */
:root {
  --blaa: #7690C1;        /* Blue - primary */
  --lilla: #B8A6C0;       /* Purple - relations */
  --groen: #8B9A9D;       /* Green/gray - elements */
  
  /* Element Colors (5 elementer) */
  --vand: #4A90A4;        /* Water - blue */
  --trae: #5B8C5A;        /* Wood - green */
  --ild: #C85A54;         /* Fire - red */
  --jord: #B8956A;        /* Earth - golden brown */
  --metal: #8B9A9D;       /* Metal - gray (same as groen) */
  
  /* UI Colors */
  --bg-primary: #FFFFFF;
  --bg-secondary: #F8F8F8;
  --bg-highlight: #F0F4F8;
  --text-primary: #333333;
  --text-secondary: #666666;
  --text-light: #999999;
  --border: #E0E0E0;
  
  /* Gradients */
  --gradient-header: linear-gradient(135deg, var(--blaa) 0%, var(--groen) 100%);
  --gradient-nav: linear-gradient(135deg, var(--groen) 0%, var(--blaa) 100%);
}
```

### Typography
```css
:root {
  /* System fonts - no external fonts */
  --font-system: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  
  /* Font Sizes */
  --text-h1: 24px;
  --text-h2: 20px;
  --text-h3: 18px;
  --text-body: 15px;
  --text-small: 13px;
  --text-tiny: 11px;
  
  /* Font Weights */
  --weight-regular: 400;
  --weight-medium: 500;
  --weight-semibold: 600;
  --weight-bold: 700;
  
  /* Line Heights */
  --line-tight: 1.2;
  --line-normal: 1.5;
  --line-relaxed: 1.8;
}
```

### Spacing
```css
:root {
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  --space-xl: 32px;
  --space-xxl: 40px;
}
```

### Layout
```css
:root {
  /* Header */
  --header-height: 60px;
  
  /* Bottom Navigation */
  --nav-height: 70px;
  
  /* Safe Areas (for iOS notch, etc.) */
  --safe-top: env(safe-area-inset-top);
  --safe-bottom: env(safe-area-inset-bottom);
  
  /* Content Area */
  --content-max-width: 600px;
  
  /* Border Radius */
  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;
  --radius-full: 9999px;
}
```

---

## NAVIGATION STRUCTURE

### Bottom Tab Navigation (4 buttons - always visible)

```html
<nav class="bottom-nav">
  <button data-screen="idag" class="nav-button active">
    <svg><!-- home icon --></svg>
    <span>I dag</span>
  </button>
  <button data-screen="udforskning" class="nav-button">
    <svg><!-- compass icon --></svg>
    <span>Udforskning</span>
  </button>
  <button data-screen="relationer" class="nav-button">
    <svg><!-- hearts icon --></svg>
    <span>Relationer</span>
  </button>
  <button data-screen="favoritter" class="nav-button">
    <svg><!-- star icon --></svg>
    <span>Favoritter</span>
  </button>
</nav>
```

### Screen Management
```javascript
// Simple routing without framework
const screens = {
  'idag': 'screens/idag.html',
  'udforskning': 'screens/udforskning.html',
  'relationer': 'screens/relationer.html',
  'favoritter': 'screens/favoritter.html'
};

function loadScreen(screenName) {
  // Fetch and inject HTML
  // Update active nav state
  // Save current screen to localStorage
}
```

---

## CORE FEATURES & CALCULATIONS

### 1. CYCLE CALCULATIONS (cycles.js)

**Life Phase (Livsfase):**
```javascript
// Women's 7-year cycle
function calculateLifePhase(age) {
  const phaseNumber = Math.floor(age / 7) + 1;
  const clampedPhase = Math.min(phaseNumber, 9);
  
  const PHASE_DATA = {
    1: { element: 'VAND', startAge: 0, endAge: 7, name: 'Livets begyndelse' },
    2: { element: 'VAND', startAge: 7, endAge: 14, name: 'Udforskning' },
    3: { element: 'TRÆ', startAge: 14, endAge: 21, name: 'Forvandling' },
    4: { element: 'TRÆ', startAge: 21, endAge: 28, name: 'Blomstring' },
    5: { element: 'ILD', startAge: 28, endAge: 35, name: 'Ansvar' },
    6: { element: 'JORD', startAge: 35, endAge: 42, name: 'Modning' },
    7: { element: 'JORD', startAge: 42, endAge: 49, name: 'Høst' },
    8: { element: 'METAL', startAge: 49, endAge: 56, name: 'Frigørelse' },
    9: { element: 'VAND', startAge: 56, endAge: 63, name: 'Visdom' }
  };
  
  return PHASE_DATA[clampedPhase];
}

// Men's 8-year cycle (for partners)
function calculateMalePhase(age) {
  const phaseNumber = Math.floor(age / 8) + 1;
  // Similar structure but 8-year intervals
  // Returns same format as female phases
}
```

**Season (Årstid):**
```javascript
function calculateSeason(date) {
  const month = date.getMonth() + 1;
  const day = date.getDate();
  
  // Danish seasons (Northern Hemisphere)
  if ((month === 12 && day >= 21) || month === 1 || month === 2 || (month === 3 && day < 21)) {
    return { season: 'Vinter', element: 'VAND' };
  }
  if ((month === 3 && day >= 21) || month === 4 || month === 5 || (month === 6 && day < 21)) {
    return { season: 'Forår', element: 'TRÆ' };
  }
  if ((month === 6 && day >= 21) || month === 7 || (month === 8 && day < 23)) {
    return { season: 'Sommer', element: 'ILD' };
  }
  if ((month === 7 && day >= 23) || (month === 8 && day < 23)) {
    return { season: 'Sensommer', element: 'JORD' }; // Overlaps with summer
  }
  if ((month === 9 && day >= 23) || month === 10 || month === 11 || (month === 12 && day < 21)) {
    return { season: 'Efterår', element: 'METAL' };
  }
}
```

**Menstrual Cycle (Månedscyklus):**
```javascript
function calculateMenstrualDay(startDate, currentDate) {
  const diffTime = currentDate - startDate;
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24));
  const cycleDay = (diffDays % 28) + 1;
  
  // Determine week and element
  if (cycleDay <= 7) {
    return { week: 1, day: cycleDay, element: 'VAND', phase: 'Menstruation' };
  } else if (cycleDay <= 14) {
    return { week: 2, day: cycleDay, element: 'TRÆ', phase: 'Follikulær' };
  } else if (cycleDay <= 21) {
    return { week: 3, day: cycleDay, element: 'ILD', phase: 'Ægløsning' };
  } else {
    return { week: 4, day: cycleDay, element: 'JORD', phase: 'Luteal' };
  }
}
```

**Weekday (Ugedag):**
```javascript
function calculateWeekday(date) {
  const WEEKDAY_DATA = {
    0: { day: 'Søndag', element: 'VAND' },
    1: { day: 'Mandag', element: 'VAND' },
    2: { day: 'Tirsdag', element: 'TRÆ' },
    3: { day: 'Onsdag', element: 'ILD' },
    4: { day: 'Torsdag', element: 'JORD' },
    5: { day: 'Fredag', element: 'METAL' },
    6: { day: 'Lørdag', element: 'JORD' }
  };
  
  return WEEKDAY_DATA[date.getDay()];
}
```

**Organ Clock (Organur):**
```javascript
function calculateOrganClock(time) {
  const hour = time.getHours();
  
  const ORGAN_CLOCK = {
    1: { organ: 'Lever', element: 'TRÆ', hours: '01-03' },
    3: { organ: 'Lunger', element: 'METAL', hours: '03-05' },
    5: { organ: 'Tyktarm', element: 'METAL', hours: '05-07' },
    7: { organ: 'Mave', element: 'JORD', hours: '07-09' },
    9: { organ: 'Milt', element: 'JORD', hours: '09-11' },
    11: { organ: 'Hjerte', element: 'ILD', hours: '11-13' },
    13: { organ: 'Tyndtarm', element: 'ILD', hours: '13-15' },
    15: { organ: 'Blære', element: 'VAND', hours: '15-17' },
    17: { organ: 'Nyrer', element: 'VAND', hours: '17-19' },
    19: { organ: 'Perikardium', element: 'ILD', hours: '19-21' },
    21: { organ: 'Tredobbelt varmer', element: 'ILD', hours: '21-23' },
    23: { organ: 'Galdeblære', element: 'TRÆ', hours: '23-01' }
  };
  
  // Find which 2-hour window we're in
  const organHour = Math.floor(hour / 2) * 2 + 1;
  return ORGAN_CLOCK[organHour];
}
```

### 2. DATA STORAGE (storage.js)

```javascript
const Storage = {
  // User profile
  saveUser(userData) {
    localStorage.setItem('user', JSON.stringify(userData));
  },
  
  getUser() {
    const data = localStorage.getItem('user');
    return data ? JSON.parse(data) : null;
  },
  
  // Relations
  saveRelations(relations) {
    localStorage.setItem('relations', JSON.stringify(relations));
  },
  
  getRelations() {
    const data = localStorage.getItem('relations');
    return data ? JSON.parse(data) : [];
  },
  
  // Favorites
  addFavorite(type, item) {
    const favorites = this.getFavorites();
    if (!favorites[type]) favorites[type] = [];
    favorites[type].push(item);
    localStorage.setItem('favorites', JSON.stringify(favorites));
  },
  
  getFavorites() {
    const data = localStorage.getItem('favorites');
    return data ? JSON.parse(data) : { exercises: [], food: [], themes: [], phases: [] };
  },
  
  // Clear all data
  clearAll() {
    localStorage.clear();
  }
};
```

### 3. PWA SETUP

**manifest.json:**
```json
{
  "name": "De 9 Livsfasers Energi",
  "short_name": "9 Livsfaser",
  "description": "Companion app til bogen om kvinders udvikling gennem ni syv-års-cyklusser",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#FFFFFF",
  "theme_color": "#7690C1",
  "orientation": "portrait",
  "lang": "da-DK",
  "icons": [
    {
      "src": "assets/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "assets/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
}
```

**Service Worker (sw.js):**
```javascript
const CACHE_NAME = 'livsfaser-v1';
const urlsToCache = [
  '/',
  '/index.html',
  '/css/variables.css',
  '/css/base.css',
  '/css/components.css',
  '/css/screens.css',
  '/js/app.js',
  '/js/cycles.js',
  '/js/storage.js',
  '/js/navigation.js',
  '/js/utils.js',
  '/screens/idag.html',
  '/screens/udforskning.html',
  '/screens/relationer.html',
  '/screens/favoritter.html'
];

// Install - cache files
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(urlsToCache))
  );
});

// Fetch - serve from cache, fallback to network
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => response || fetch(event.request))
  );
});

// Activate - clean old caches
self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheName !== CACHE_NAME) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

---

## ISABELLE'S VOICE - CRITICAL

**Dagens Tema** must sound like Isabelle wrote it - NOT generic TCM:

❌ BAD: "Metal element is strong today. Focus on letting go."
✅ GOOD: "Når Metal står stærk, viser det essentielle sig. Giv slip på det overflødige."

**Characteristics:**
- Poetisk, personlig
- 15-30 word sentences with natural flow
- Uses "måske", "kan", "jeg har selv oplevet"
- NEVER "det er vigtigt at", "husk at", "man skal"
- Natural Danish discourse particles: "nemlig", "jo", "faktisk"

**Store content as JSON with this voice!**

---

## INCLUSIVITY - CRITICAL

**This app is for ALL women:**
- With children
- Without children
- In relationships
- Single
- Any life path

**Onboarding must state:**
"Denne app er for ALLE kvinder - med børn, uden børn, i parforhold eller alene. Faserne angår os alle."

**Relations are OPTIONAL:**
App functions fully without adding any relations.
Main focus is on USER HERSELF first.

---

## DEVELOPMENT PRIORITIES

### MVP (Phase 1) - MUST HAVE
1. ✅ PWA setup (manifest, service worker)
2. ✅ Basic navigation (4 tabs working)
3. ✅ User age input → calculate life phase
4. ✅ "I dag" screen showing all 5 cycles (text only first)
5. ✅ Cycle calculations working correctly
6. ⏳ Basic phase content (text from book)
7. ⏳ LocalStorage working (save user data)

### Phase 2 - IMPORTANT
1. Concentric circles visualization (CSS/SVG)
2. Add one relation (partner or child)
3. Basic matching (show 2 people's phases)
4. Favorites system
5. Complete phase content

### Phase 3 - NICE TO HAVE
1. Date analysis (FORTID/NUTID/FREMTID)
2. Multiple relations (3-4 people)
3. Tracking features
4. Graphs and visualizations

---

## PERFORMANCE REQUIREMENTS

- First load: < 2 seconds
- Screen transitions: < 300ms
- Offline-first: Core features work without internet
- Mobile-first: Designed for phones, works on desktop
- PWA installable on iOS and Android

---

## BROWSER SUPPORT

- Chrome/Edge 90+
- Safari 14+ (iOS 14+)
- Firefox 88+
- Samsung Internet 14+

---

## DEPLOYMENT

**GitHub Pages:**
```bash
# Simple deployment
git add .
git commit -m "Update app"
git push origin main

# Served from: https://username.github.io/livsfaser-app/
```

**No build step required!** Just push and it's live.

---

## CRITICAL REMINDERS

1. **NO FRAMEWORKS** - Pure HTML/CSS/JS only
2. **NO BUILD PROCESS** - Direct deployment
3. **Men's 8-year cycle** ⭐ Explain clearly when adding partner
4. **Isabelle's voice** - All content sounds personal, not clinical
5. **Accurate calculations** - Cycle math must be perfect
6. **Inclusive** - For ALL women (state this clearly)
7. **Mobile-first** - Design for phone, enhance for desktop
8. **Offline-capable** - Service worker essential

---

## USEFUL COMMANDS

```bash
# Local development (simple HTTP server)
python3 -m http.server 8000
# or
npx http-server -p 8000

# Open in browser
open http://localhost:8000

# No build, no compile, no dependencies!
```

---

## REFERENCES

- Sitemap: /outputs/sitemap_komplet.md
- Design mockups: /outputs/*.html (29 HTML screens)
- Book structure: /project/de_9_livsfasers_energi_v4_komplet.docx

---

*This is your brain for building a PWA. Read it before starting development.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nklasfff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nklasfff)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
