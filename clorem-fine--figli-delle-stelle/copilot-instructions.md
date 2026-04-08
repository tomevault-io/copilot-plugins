## figli-delle-stelle

> > **AI Coding Agent Instructions**

# AGENTS.md - Figli delle Stelle

> **AI Coding Agent Instructions**
> 
> This file contains essential information about the project structure, technology stack, and development guidelines. Read this first before making any changes.

---

## Project Overview

**Figli delle Stelle** è un sito web statico per la promozione di eventi esclusivi organizzati a Lucca, Italia. Il progetto attuale è focalizzato sull'evento "Pasquetta 2026" che si terrà il 6 Aprile 2026 all'Antico Caffè delle Mura.

### Pagine Principali

| File | Descrizione |
|------|-------------|
| `index.html` | Homepage con logo animato, carosello foto e lista eventi |
| `evento-pasquetta.html` | Pagina dettaglio evento con form prenotazione e chat assistant |

---

## Technology Stack

### Core Technologies
- **HTML5** - Markup semantico
- **CSS3** - Con variabili CSS, animazioni, backdrop-filter (glassmorphism)
- **JavaScript Vanilla** - Nessun framework, codice puro ES6+

### External Dependencies (CDN)
| Libreria | CDN | Uso |
|----------|-----|-----|
| tsParticles | `cdn.jsdelivr.net/npm/tsparticles@2.12.0` | Effetto stelle animate (solo homepage) |
| Font Awesome | `cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0` | Icone |
| Google Fonts | `fonts.googleapis.com` | Playfair Display, Cormorant Garamond, Inter |

### Nota importante
**Nessun package manager** - Il progetto non utilizza npm, yarn o altri gestori di pacchetti. Tutte le dipendenze sono caricate via CDN. Non esiste `package.json`.

---

## Project Structure

```
v.0/
├── index.html                  # Homepage
├── evento-pasquetta.html       # Pagina evento Pasquetta 2026
├── README.md                   # Documentazione utente
├── AGENTS.md                   # Questo file
├── screenshot.js               # Script Puppeteer per screenshot (Node.js)
├── css/
│   ├── style.css              # Stili base, variabili CSS, componenti condivisi
│   ├── home.css               # Stili specifici homepage
│   └── evento.css             # Stili specifici pagina evento
├── js/
│   ├── main.js                # JS principale: particelle, navigazione, form
│   ├── home.js                # JS homepage: tsParticles, carosello
│   ├── whatsapp-booking.js    # Gestione prenotazioni via WhatsApp
│   └── chat-assistant.js      # Chatbot per FAQ automatiche
└── assets/
    └── images/                # Logo SVG e foto eventi (JPG/PNG)
```

---

## CSS Architecture

### File Organization

1. **`css/style.css`** (1223 righe) - Stili fondamentali
   - Variabili CSS con tema blu (`--color-accent: #3b82f6`)
   - Reset e base styles
   - Navigazione (pill style)
   - Componenti: bottoni, card pacchetti, form, modal, footer
   - Animazioni keyframes
   - Responsive breakpoints

2. **`css/home.css`** (719 righe) - Stili homepage
   - Hero con logo animato
   - Effetto particelle logo
   - Carosello orizzontale draggabile
   - Card eventi stile Apple TV
   - Sezione Instagram

3. **`css/evento.css`** (1000+ righe) - Stili pagina evento
   - Hero con gradiente blu/rosso
   - Collaboration logos (Figli delle Stelle × Solea)
   - Message box chat-style
   - Location section con mockup iPhone
   - DJ lineup cards (mystery reveal)

### Variabili CSS Principali

```css
:root {
    --color-bg: #050a14;           /* Sfondo principale */
    --color-accent: #3b82f6;        /* Blu accent */
    --color-accent-light: #60a5fa;  /* Blu chiaro */
    --font-display: 'Playfair Display', serif;
    --font-body: 'Inter', sans-serif;
    --transition-medium: 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}
```

### Pattern CSS Utilizzati
- **Glassmorphism**: `backdrop-filter: blur(20px) saturate(180%)`
- **Gradienti**: Da blu (`#3b82f6`) a rosso (`#dc2626`) nella pagina evento
- **Responsive**: Mobile-first con breakpoint a 992px, 768px, 480px

---

## JavaScript Architecture

### File Organization

1. **`js/main.js`** (527 righe) - Core functionality
   - `initParticles()` - Particelle animate custom (non tsParticles)
   - `initNavigation()` - Navbar scroll effect, mobile toggle
   - `initScrollReveal()` - Animazioni al scroll
   - `initFormHandling()` - Gestione form prenotazione
   - `initPackageSelection()` - Selezione pacchetto
   - `initModal()` - Modal successo
   - Integrazione WhatsApp per prenotazioni

2. **`js/home.js`** (414 righe) - Homepage specific
   - `initStarsBackground()` - tsParticles configurazione
   - `initHorizontalCarousel()` - Carosello infinito draggabile
   - `initLogoParticles()` - Particelle attorno al logo
   - Parallax effect on scroll
   - IntersectionObserver per animazioni

3. **`js/whatsapp-booking.js`** (126 righe) - Prenotazioni WhatsApp
   - Gestione standalone del form prenotazione
   - Calcolo prezzi dinamico
   - Apertura WhatsApp con messaggio precompilato
   - Numero WhatsApp hardcoded: `393450688943`

4. **`js/chat-assistant.js`** (271 righe) - Chatbot FAQ
   - Knowledge base con risposte automatiche
   - Keyword matching per: prezzi, orari, location, pacchetti, etc.
   - UI chat stile message bubbles

### Pattern JS Utilizzati
- **Modular functions**: Ogni feature inizializzata separatamente
- **DOMContentLoaded**: Tutti gli script aspettano il DOM ready
- **Feature detection**: Controllo esistenza elementi prima di operare
- **Event delegation**: Per elementi dinamici

---

## Features Key

### 1. Sistema di Prenotazione WhatsApp
Il form prenotazione (`#bookingForm`) invia i dati via WhatsApp invece di un backend:

```javascript
// Flusso:
1. Utente compila form (nome, telefono, persone, pacchetto)
2. Calcolo totale: €20 (pranzo) o €10 (dopo-pranzo) × numero persone
3. Generazione URL WhatsApp con messaggio precompilato
4. Apertura WhatsApp Web/App in nuova tab
```

**File coinvolti:**
- `evento-pasquetta.html` - Form HTML
- `js/whatsapp-booking.js` - Logica principale
- `js/main.js` - Logica alternativa (fallback)

**Personalizzazione prezzi:**
```javascript
// js/whatsapp-booking.js riga 22-25
const prices = {
    'pranzo': 20,
    'dopo-pranzo': 10
};
```

### 2. Chat Assistant
Sistema FAQ automatico nella hero section:
- Input textarea per domande
- Quick action buttons (Prezzi, Orari, Location, Pacchetti)
- Risposte automatiche basate su keyword matching
- Knowledge base in `js/chat-assistant.js`

### 3. DJ Lineup Mystery Reveal
Sezione "Line Up" con card DJ:
- Avatar placeholder con icona "user-secret"
- Click per rivelare (classe CSS `revealed`)
- Progress dots per tracking
- **Nota**: I nomi reali dei DJ non sono ancora stati aggiunti

### 4. Carosello Homepage
Carosello foto orizzontale in `index.html`:
- Scroll automatico continuo
- Draggabile con mouse/touch
- Loop infinito (card clonate)
- Pause on hover

---

## Development Guidelines

### Convenzioni Codice

#### CSS
- Usare **variabili CSS** da `:root` per colori e spaziature
- **Mobile-first**: Scrivere per mobile, aggiungere media query per desktop
- **BEM-like naming**: `.component-name`, `.component-element`
- **Glassmorphism**: Usare sempre `backdrop-filter` con fallback

#### JavaScript
- **Feature detection** prima di manipolare elementi:
  ```javascript
  const element = document.getElementById('id');
  if (!element) return; // Guard clause
  ```
- **Event delegation** per elementi dinamici
- **No jQuery** - Usare API DOM native
- **Template literals** per HTML dinamico

#### HTML
- **Attributi data-** per configurazione: `data-package="pranzo"`
- **ID univoci** per elementi interattivi
- **Classi CSS** per styling, **ID** per JS hooks

### Modifica Prezzi
Per cambiare i prezzi dell'evento, modificare in **tre posti**:
1. `js/whatsapp-booking.js` riga 22-25 (logica)
2. `js/main.js` riga 247-250 (logica alternativa)
3. `evento-pasquetta.html` testo visibile nelle card pacchetti

### Aggiunta Nuovi Eventi
Per aggiungere un nuovo evento alla homepage:
1. Aggiungere card in `index.html` sezione `#eventi`
2. Classe `event-card-modern` per stile Apple TV
3. `featured` per evento principale, `coming-soon` per futuri
4. Aggiungere immagine in `assets/images/` o URL esterno

---

## Testing & Debug

### Visualizzazione Locale
Aprire direttamente i file HTML nel browser:
```bash
# Mac
open index.html

# O usare un server locale
python3 -m http.server 8000
```

### Checklist Modifiche
- [ ] Test responsive su mobile (375px), tablet (768px), desktop (1440px)
- [ ] Verificare che tsParticles funzioni su homepage
- [ ] Test form prenotazione (verifica URL WhatsApp generato in console)
- [ ] Test chat assistant con varie domande
- [ ] Verificare contrasti colori per accessibilità

### Debug Tips
- **Console**: Tutti gli script loggano messaggi iniziali
- **WhatsApp URL**: `console.log('Opening:', whatsappURL)` in main.js
- **tsParticles**: Se non visibile, controllare errore in console (dipendenza CDN)

---

## Deployment

### Hosting
Sito statico, può essere deployato su:
- **GitHub Pages** - Gratuito, push su branch gh-pages
- **Netlify** - Drag & drop cartella
- **Vercel** - Connessione GitHub
- **SiteGround** - Upload FTP (skill disponibile)

### Pre-deploy Checklist
- [ ] Rimuovere riferimenti locali da `screenshot.js` (se usato)
- [ ] Verificare che tutti i link siano relativi (`./` non `/`)
- [ ] Comprimere immagini se necessario (molte immagini sono >5MB)
- [ ] Testare form prenotazione con numero WhatsApp corretto

---

## Assets

### Immagini
Cartella `assets/images/`:
- `figli delle stelle .svg` - Logo principale
- `9.svg` - Logo Solea (collaborazione)
- `Figli delle stelle_*.jpg` - Foto eventi per carosello
- Total: ~520MB di asset immagini

### Font
Caricati via Google Fonts:
- Playfair Display (titoli)
- Cormorant Garamond (accenti eleganti)
- Inter (testo body)

---

## Note per Agenti AI

### Cose da Sapere
1. **Nessun build step** - Modifiche immediate sui file
2. **No framework** - JavaScript vanilla, niente React/Vue/Angular
3. **Italiano** - Tutto il contenuto è in italiano
4. **Hardcoded values** - Numeri WhatsApp, date, prezzi sono hardcoded

### Cose da NON Fare
- NON aggiungere package.json o dipendenze npm
- NON modificare la struttura cartelle senza aggiornare i path
- NON rimuovere i fallback per WhatsApp (popup blockers)
- NON cambiare classe CSS `event-card-modern` senza verificare lo stile

### Estensioni Comuni
- **Nuovo evento**: Copiare struttura da `evento-pasquetta.html`
- **Nuova pagina**: Includere `style.css` + CSS specifico
- **Nuovo componente JS**: Aggiungere funzione `initXxx()` in main.js
- **Nuovo font**: Aggiungere a Google Fonts link in `<head>`

---

## Troubleshooting

| Problema | Causa | Soluzione |
|----------|-------|-----------|
| tsParticles non funziona | CDN non caricato | Verificare connessione internet, controllare console errori |
| WhatsApp non si apre | Popup blocker | Il codice ha fallback a `window.location.href` |
| Form non calcola totale | JS non caricato | Verificare ordine script in HTML, controllare console |
| Card non animate | IntersectionObserver | Verificare che elementi abbiano classe `event-card` |
| Mobile menu non funziona | CSS/JS mismatch | Controllare che `navToggle` e `navMenu` esistano |

---

## Contatti & Riferimenti

- **Instagram**: @___figlidellestelle___
- **Location**: Antico Caffè delle Mura, Lucca
- **Data evento**: Lunedì 6 Aprile 2026
- **WhatsApp prenotazioni**: +39 345 068 8943

---

*Ultimo aggiornamento: Marzo 2026*
*Progetto: Figli delle Stelle - Pasquetta 2026*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CloreM-Fine)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CloreM-Fine)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
