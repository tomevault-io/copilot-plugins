## blog

> - Ogni nuovo articolo deve essere creato in `trading/blog/<slug>/index.html`.

# Istruzioni per l'agente — Zenit Blog

## Struttura URL obbligatoria

- Ogni nuovo articolo deve essere creato in `trading/blog/<slug>/index.html`.
- L'URL pubblico deve essere `https://zenitcoach.com/trading/blog/<slug>/`, con trailing slash e senza estensione `.html`.
- La cartella `blog/` contiene esclusivamente pagine ponte legacy con `noindex, follow`, canonical e redirect verso il nuovo URL.
- Non pubblicare mai nuovi articoli completi in `blog/*.html`.
- L'autore degli articoli è `Edoardo` (`Person` nello schema Article); il publisher rimane `Zenit Coaching` (`Organization`).

## Quando si pubblica o modifica un articolo

Ogni volta che viene creato o modificato un file in `trading/blog/*/index.html`, **devi** eseguire questi passaggi prima di committare:

1. **Aggiornare il blog index**
   ```bash
   python3 scripts/update-blog-index.py
   ```
   Questo sincronizza `trading/blog/index.html` esclusivamente con gli articoli presenti in `trading/blog/*/index.html`.

2. **Verificare GTM (Google Tag Manager)**
   Lo script sopra segnala se manca GTM in qualche pagina. In caso, incolla in **<head>** e subito dopo **<body>** di ogni nuovo file HTML gli snippet forniti da Google Tag Manager (ID `GTM-MCBW9JTG`).

3. **Rigenerare sitemap e robots**
   ```bash
   python3 scripts/generate-seo-files.py
   ```

4. **Verificare URL e dominio**
   Tutti i link, canonical, Open Graph e schema devono usare `https://zenitcoach.com/trading/blog/<slug>/`, **mai** `zenitcoaching.it`, `/blog/*.html` o URL senza trailing slash.

5. **Committare e pushare**
   Includi sempre articolo, blog index e sitemap aggiornati nello stesso commit.

---
> Source: [ZenitCoaching/blog](https://github.com/ZenitCoaching/blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
