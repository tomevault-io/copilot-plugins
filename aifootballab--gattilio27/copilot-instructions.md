## gattilio27

> UX e responsività obbligatorie; il prodotto è una guida per l'utente (primi al mondo).


# UX e responsività (obbligatorie)

- **Siamo una guida.** Ogni schermata deve guidare l'utente: contesto chiaro (titolo/intro), "cosa faccio qui", prossimo passo e CTA evidenti. Nessuna pagina "muta".
- **Responsività obbligatoria.** Ogni layout e componente deve funzionare da ~320px a 1400px: usare `clamp()` per font e spacing, `minWidth: 0` su flex/grid children, `flexWrap` dove serve. Su mobile (&lt; 480px): barra/contenuto principale full width, bottoni secondari sotto; touch target ≥ 44px.
- **Riferimento:** seguire `docs/UX_RESPONSIVE_LINEE_GUIDA.md` (breakpoint, modali, tabelle, checklist) e i principi in `docs/PIANO_UX_ENTERPRISE_GUIDA_CLIENTE.md` (§0).
- **Verifica:** niente overflow orizzontale; gerarchia visiva corretta su mobile; testi guida e i18n coerenti.
- **Mostra altro / Guida:** sezioni espandibili (Ultime partite, Obiettivi, Guide per pagina) con stesso pattern: `role="button"`, `aria-expanded`, `aria-label` (expandSection*/collapseSection* o expandGuideCard/collapseGuideCard), tastiera Enter/Space, touch target ≥ 44px. Link "Guida" sempre con `guideLink` verso `/guida`. Vedi §2.7 e §2.8 in `docs/UX_RESPONSIVE_LINEE_GUIDA.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aifootballab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aifootballab)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
