## gxcode

> - **Skills**: Invocabili via terminale con `gx-skill run "<nome>"`

# PROJECT GUIDELINES & TOOLS (GXCode)

## STANDARDS & COMMANDS
- **Skills**: Invocabili via terminale con `gx-skill run "<nome>"`
- **Agenti**: Info dettagliate in `.claudecode/GX_IDENTITY.md`

### AVAILABLE SKILLS
- `Brief Analyzer`: Analizza il brief del sito, estrae obiettivi, target, tono, vincoli e segnala eventuali informazioni mancanti prima della progettazione del mockup.
- `Creative Direction Builder`: Definisce la direzione visiva del mockup: mood, palette, tipografia, stile UI e tone of voice, coerenti con brand, target e obiettivo.
- `LangChain Core`: Core framework for developing applications powered by language models.
- `Layout Architect`: Progetta la struttura della pagina e organizza le sezioni del mockup in un ordine strategico, in base a obiettivo, contenuti e tipo di sito.
- `Responsive Mockup Planner`: Adatta il mockup alla visualizzazione mobile e tablet, definendo il comportamento responsive delle sezioni e la priorità dei contenuti.
- `UI Copy Generator`: Genera headline, subheadline, CTA, microcopy e testi chiave del mockup, mantenendo coerenza con target, prodotto e tone of voice.
- `Wireframe Writer`: Converte la struttura del sito in un wireframe testuale dettagliato, descrivendo il layout di ogni sezione in modo chiaro per designer e sviluppatori.

## MODEL SELECTION STRATEGY
Per ottimizzare costi e prestazioni, Claude deve seguire queste linee guida nella scelta del modello:
- **Claude 3 Haiku**: Analisi di file esistenti, analisi del brief di progetto (es: `Brief Analyzer`) e compiti di sola lettura.
- **Claude 3.5 Sonnet**: Generazione di nuovo codice, creazione di layout e documenti (es: `UI Copy Generator`, `Layout Architect`), applicazione di modifiche e refactoring.
- **Claude 3 Opus**: Gestione di agenti complessi, delega di compiti a skill esterne e creazione/orchestrazione di sotto-agenti specializzati.

## CURRENT WORKSPACE CONTEXT
- **Root Path**: ./ (Current project root)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kyriga-CGX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
