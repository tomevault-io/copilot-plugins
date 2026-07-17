## longkai2

> Codex deve lavorare come agente tecnico sulla repository, leggendo i file necessari, proponendo un piano e applicando solo le modifiche richieste.

# AGENTS.md 

## Obiettivo

Codex deve lavorare come agente tecnico sulla repository, leggendo i file necessari, proponendo un piano e applicando solo le modifiche richieste.

## Regole obbligatorie

- Prima di modificare qualsiasi file, leggere i file rilevanti.
- Prima di applicare modifiche, spiegare brevemente il piano.
- Modificare solo i file necessari al task.
- Non modificare file non richiesti.
- Non fare refactor generale.
- Non cambiare testi, SEO, title, meta description, canonical, JSON-LD, URL, immagini, email, numeri di telefono o link interni se non richiesto.
- Non inventare percorsi, immagini, dati aziendali o contenuti.
- Non cambiare nomi classi se non indispensabile.
- Non eliminare codice esistente senza spiegare il motivo.
- Preferire modifiche minime, robuste e facili da verificare.

## Internet

- Non usare internet se non è necessario.
- Se serve internet, fermarsi prima e chiedere approvazione.
- Prima di chiedere internet, spiegare:
  1. perché serve;
  2. quale comando o azione verrebbe eseguita;
  3. quali risorse o domini servono;
  4. cosa succede se non viene concesso accesso.
- Non installare pacchetti o dipendenze senza approvazione esplicita.
- Non aggiungere CDN, script esterni o librerie senza richiesta esplicita.

## Git

- Non lavorare direttamente su main.
- Prima di modificare, controllare il branch attuale.
- Non fare commit senza richiesta esplicita.
- Non fare push senza richiesta esplicita.
- Non fare merge senza richiesta esplicita.
- Alla fine mostrare i file modificati e il riepilogo del diff.

## CSS

- Prima di modificare CSS, controllare:
  - ordine dei file CSS caricati;
  - media query;
  - regole duplicate;
  - specificità;
  - classi presenti nell’HTML;
  - eventuali !important già presenti.
- Non aggiungere CSS inutile.
- Non creare nuove classi se si può risolvere con quelle esistenti.
- Se il problema è mobile, non modificare il layout desktop se non necessario.

## HTML

- Mantenere struttura semantica.
- Non modificare header, footer, form, script o schema se non richiesto.
- Non cambiare percorsi asset.
- Non cambiare H1, title, meta description o JSON-LD se non richiesto.

## JavaScript

- Non rompere funzioni esistenti.
- Non introdurre librerie esterne senza richiesta.
- Se possibile, controllare errori evidenti prima di modificare.

## Output finale

Alla fine di ogni task restituire:

1. file letti;
2. file modificati;
3. riepilogo modifiche;
4. eventuali rischi;
5. controlli manuali da fare nel browser.

---
> Source: [hsagency587/LONGKAI2](https://github.com/hsagency587/LONGKAI2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
