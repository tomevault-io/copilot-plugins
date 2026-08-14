## my-portfolio

> 1. Leggi questo file per intero e controlla `git status --short`: non sovrascrivere modifiche non correlate.

# Istruzioni per gli agenti — Portfolio

## Prima di modificare il progetto

1. Leggi questo file per intero e controlla `git status --short`: non sovrascrivere modifiche non correlate.
2. Leggi i componenti e gli stili coinvolti prima di cambiare il comportamento.
3. Leggi `docs/REWORK-PLAN.md` per comprendere la direzione del rework e `docs/ANALISI-STATO-ATTUALE.md` per il contesto.
4. Per ogni modifica, verifica coerenza con `UI_GUIDELINES.md` (nella root del workspace).
5. Al termine esegui `npm run lint` e `npm run build`.

## Stack e comandi

- Next.js 16 (App Router) + React 19, JavaScript JSX e CSS Modules.
- Entry point: `src/app/layout.js`; pagina: `src/app/page.js`.
- Avvio locale: `npm run dev`; lint: `npm run lint`; build: `npm run build`.
- Non aggiungere dipendenze senza una motivazione concreta. Il progetto usa CSS Modules nativi di Next.js, non Tailwind.

## Architettura da mantenere

Il progetto adotta una struttura a componenti ispirata ad Atomic Design:

- `src/components/atoms/`: componenti visuali di base riutilizzabili (Button, SectionTitle, SkillCard, SocialLink).
- `src/components/organisms/`: sezioni autonome della pagina (Navbar, Hero, About, Skills, Projects, Gallery, Contact, Footer).
- `src/components/layout/`: componenti di layout (Container).
- `src/hooks/`: logica riutilizzabile (useScrollReveal, useActiveSection, useSmoothScroll).
- `src/lib/`: utility generiche (cn per classi CSS).
- `src/data/`: dati statici estratti dai componenti (skills, projects, social, photos).
- `public/`: asset statici serviti direttamente.

Flusso dati previsto: `page.js` compone le sezioni; ogni organismo è autonomo e riceve dati da `src/data/` o dal proprio modulo. I componenti visuali ricevono solo props. Non duplicare dati o stato tra componenti.

## Convenzioni JavaScript e React

- Usa componenti funzionali, hook e `export default` in modo coerente.
- Una responsabilità per componente. Se un componente supera le 80 righe di JSX, valuta di splittarlo.
- CSS Modules per ogni componente: il file `.module.css` vive accanto al `.jsx` con lo stesso nome.
- `globals.css` contiene solo reset, token CSS (`:root`) e utility minime (`.sr-only`).
- Usa `next/image` per tutte le immagini con `width`, `height`, `alt` e `sizes` appropriati.
- Usa `next/font` per i font, non `<link>` esterni.
- Usa `next/link` per la navigazione interna.
- Non usare indici come chiavi React. Usa identificatori stabili (slug, id).
- Mantieni la formattazione coerente con il file che modifichi.

## Design System

- Token CSS definiti in `globals.css` su `:root`. Consumali ovunque, mai hardcoded.
- Palette: bg `#F8F8F6`, testo `#111111`, testo secondario `#666666`, bordi `#E8E8E8`.
- Font: Inter (via `next/font/google`).
- Tipografia: hero 72–96px, section title 48–64px, subtitle 22–28px, body 18px, caption 14px.
- Spazi: padding desktop 80px, tablet 48px, mobile 20px. Gap sezioni 120–180px.
- Radius: 8px (small), 16px (medium), 24px (large).
- Transizioni: hover 200–300ms, reveal 700–900ms, easing easeOutQuart o easeOutExpo.
- Hover: scale, opacity, translate. Mai rotate, bounce, elastic.
- Card hover: scale(1.01) o translateY(-4px), mai più aggressivo.

## HTML, accessibilità e CSS

- Usa elementi semantici (`header`, `main`, `footer`, `section`, `nav`, `article`).
- Ogni controllo cliccabile deve essere un `<button>` o un `<a>`, mai un `div` con `onClick`.
- Tutti i controlli devono funzionare da tastiera e avere stati `:focus-visible`.
- Fornisci `alt` descrittivi alle immagini informative e `alt=""` alle immagini decorative.
- Non usare `transition: all`. Specifica sempre la proprietà.
- Rispetta `prefers-reduced-motion`: disabilita animazioni di reveal e transizioni non essenziali.
- Contrasto sufficiente (WCAG AA). Touch target minimo 44x44px.
- Ogni componente possiede il proprio file CSS Module. Non usare selettori globali per stili di componente.

## Qualità

- Prima di dichiarare completata una modifica: esegui `npm run lint` e `npm run build`, poi verifica manualmente desktop, mobile e tastiera.
- Aggiorna `docs/REWORK-PLAN.md` quando una fase viene completata.
- Aggiorna README quando cambiano setup, architettura o dipendenze.

---
> Source: [vincenzocrucitti04-hash/my-portfolio](https://github.com/vincenzocrucitti04-hash/my-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
