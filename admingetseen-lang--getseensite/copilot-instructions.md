## getseensite

> Premium, scroll-driven marketing site (Vite + React + TS + Tailwind + Framer

# GetSeen — Flagship Website

Premium, scroll-driven marketing site (Vite + React + TS + Tailwind + Framer
Motion + Lenis). See `README.md` for stack, structure and deploy notes.
Use the `@/` import alias for `src` (e.g. `@/lib/motion`, `@/components/Reveal`).

## Motion & Animation
- Nutze AUSSCHLIESSLICH die Primitives aus `src/lib/motion.ts` und die
  Komponenten `<Reveal>` / `<TextReveal>`. Keine neuen Fade-Variants inline.
- Easing: immer `[0.16, 1, 0.3, 1]` (expo-out) für Reveals. NIEMALS default
  `ease` oder `easeInOut`.
- Distanzen klein halten: y 16–24px max. Kein 60–100px Travel.
- Einmal reveal: `viewport={{ once: true }}`. Niemals beim Scrollen re-triggern.
- Dauer 0.4–0.7s. Stagger zwischen Kindern 0.06–0.08s, nie 0.15s+.
- Nur transform & opacity animieren (60fps). Kein width/height/top/left.
- Immer `useReducedMotion()` respektieren.
- Zurückhaltung: nur Section-Header und Kerninhalt animieren, nicht jedes
  Element. Dekoratives erscheint ohne Animation.
- Parallax: höchstens EIN subtiles Hero-Element, ≤15% Travel. Nicht auf jedem Bild.

---
> Source: [admingetseen-lang/getseensite](https://github.com/admingetseen-lang/getseensite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
