## avl-render-tecnici

> Workflow standard per disegni tecnici LEDwall (HTML + 2D CAD)


# Workflow Standard — Disegno Tecnico AVL

## Output sempre richiesto per ogni progetto
1. **HTML interattivo** (export via app → cartella [NomeEvento]/viewer.html):
   - Viewer 3D con Three.js r128 (CDN)
   - 4 viste preset: 3D LIBERO, FRONTALE, PIANTA, LATERALE
   - Panel sinistro: dati tecnici
   - Panel destro: legenda colori
2. **4 Canvas 2D** (disegni tecnici dentro lo stesso HTML):
   - Vista frontale (canvas grande, ~960×580)
   - Sezione laterale (~420×580)
   - Pianta top view (~640×380)
   - Sezione QX30 cross-section (~380×380)

## Struttura file
- **Template:** `avl-render/public/viewer-template.html` — template HTML per export
- **Export:** l'app genera cartella [NomeEvento]/ con viewer.html + 3 PNG

## Coordinate system (tutti i disegni)
- Unità 3D: **METRI** (Three.js)
- Unità 2D (Canvas): **MILLIMETRI** interni, poi scalati in px
- Y=0: piano terra | Z=0: fronte LED | X=0: estremo sinistro LED

## Costanti da adattare per ogni progetto (in testa all'HTML)
```javascript
const P = {
  LED_W: 5.0,        // larghezza ledwall (m)
  LED_H: 2.0,        // altezza fisica (m)
  LED_H_ACTIVE: 1.5, // altezza attiva (m)
  CAB: 0.5,          // dimensione cabinet (m)
  BOT_BAR: 0.10,     // bottom bar (m)
  LEG_X: [...],      // posizioni X gambe (m)
  LEG_H: 2.0,        // altezza gamba (m)
  LEG_ARM: 0.42,     // braccio orizzontale L (m)
  // QX30 fisso:
  QX: 0.290, CHORD_R: 0.0255, DIAG_R: 0.008,
  Z_LED_BACK: 0.08, Z_GAP: 0.15, TUBE_R: 0.025,
};
```

## Regole disegni 2D (standard professionale)
- **Sfondo**: bianco `#ffffff`
- **Linee oggetto**: nere, lineWidth 1.5–2
- **Linee quota**: blu `#0040ff`, lineWidth 1
- **Dead zone**: hatch diagonale rosso + bordo tratteggiato
- **Truss QX30**: fill `rgba(96,112,144,0.25)`, corde `#607090`, bracing `#546e7a`
- **LED attivo**: `rgba(26,111,206,...)`
- **LED spento**: `rgba(44,62,80,...)`
- **Tubi Ø50**: `#c0392b`
- **Aliscaff**: `#27ae60`
- **Bottom bar**: `#78909c`
- **Title block**: sempre in fondo al canvas (h=36px) con: TITOLO VISTA | NOME PROGETTO | SCALA | DATA

## Funzioni helper obbligatorie
```javascript
dimH(ctx, x1, x2, y, text, tY, color)  // quota orizzontale
dimV(ctx, x, y1, y2, text, tX, color)  // quota verticale
titleBlock(ctx, w, h, title, scale, project, date)
```

## Quote minime da indicare sempre
Vista frontale: larghezza totale, interasse gambe, offset bordi, H totale, H LED, H attivo, bottom bar, quota cabinet
Vista laterale: profondità LED, gap, sezione QX30, braccio L, H totale, posizioni tubi
Pianta: larghezza, interasse gambe, offset, profondità LED, sezione QX30, braccio L
Sezione QX30: 290×290 mm, Ø51 corde, Ø16 diagonali

## Materiali e colori Three.js
```javascript
const MAT = {
  chord:   MeshLambertMaterial({ color: 0x607090 }),
  diagLine: LineBasicMaterial({ color: 0x455a64 }),
  ledON:   MeshLambertMaterial({ color: 0x1a6fce, emissive: 0x0a2a6a }),
  ledOFF:  MeshLambertMaterial({ color: 0x2c3e50 }),
  bar:     MeshLambertMaterial({ color: 0x607090 }),
  tube:    MeshLambertMaterial({ color: 0xc0392b }),
  clamp:   MeshLambertMaterial({ color: 0x27ae60 }),
  base:    MeshLambertMaterial({ color: 0x444455 }),
};
```

## Checklist per ogni nuovo disegno
- [ ] Aggiornare `const P = {...}` con le nuove misure
- [ ] Aggiornare `legX` (posizioni gambe)
- [ ] Aggiornare `TUBE_Y` (altezze tubi)
- [ ] Aggiornare panel dati tecnici (HTML sinistra)
- [ ] Verificare scale 2D (px/mm) in ogni canvas
- [ ] Verificare tutte le quote
- [ ] Aggiornare titolo e data nel title block
- [ ] Testare nel browser prima di consegnare

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Andraven11)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Andraven11)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
