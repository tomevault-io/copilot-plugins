## davraf-guidelines

> - Rispondi sempre in **italiano**

# Linee guida per Claude Code

## Lingua
- Rispondi sempre in **italiano**

## Comportamento generale
- Se l'utente scrive un termine tecnico in modo errato o impreciso, segnalarlo gentilmente e fornire la forma corretta, in modo che possa imparare la terminologia giusta

## Standard di progetto .NET
@.github/copilot-instructions.md

## Regola MCP Server
@.github/instructions/mcp-server-discovery.instructions.md

## Modifiche al codice

Ogni richiesta che comporta una modifica al codice **deve essere proposta come plan** prima di essere eseguita. Non scrivere codice senza che il piano sia stato approvato esplicitamente dall'utente.

## Citazione fonti e modello

Alla fine di ogni risposta, se sono stati letti file o consultati documenti:
- Cita i file usati come fonti (path relativo)
- Indica il modello LLM usato (es. `claude-sonnet-4-6`)

## Invocazione automatica delle skill

Quando l'utente esprime un intento che corrisponde a una delle skill disponibili,
**invoca direttamente la skill** senza attendere conferma. Usa il contesto della
conversazione come argomento passato alla skill.

| Se l'utente dice qualcosa come... | Invoca |
|-----------------------------------|--------|
| "vai professor", "scrivi la doc", "aggiorna il README", "genera la scheda del progetto", "documenta gli endpoint", "prepara l'onboarding" | `/professor [richiesta]` |
| "consulta il warroom", "sentiamo le opinioni", "apri il tavolo", "cosa ne pensano gli esperti", "discutiamo questa scelta" | `/warroom [domanda o contesto]` |
| "chiedi al tattico", "rivedi questo prompt", "migliora il prompt", "scrivi un prompt per", "perché questo prompt non funziona" | `/tattico [prompt o descrizione]` |
| "pianifica il rilascio", "prepara l'ambiente", "come si deploya", "configura Docker", "procedura di deploy" | `/tech [task]` |
| "audit api", "fai l'audit del backend", "analizza le api", "cerca dead code", "controlla il codice backend" | `/audit-api [focus opzionale]` |
| "audit frontend", "fai l'audit del fe", "analizza il frontend", "controlla i componenti" | `/audit-fe [focus opzionale]` |

Quando invochi una skill, passa come argomento tutto il contesto utile già presente
nella conversazione (codice aperto, domanda originale, file citati) — non chiedere
all'utente di ripetere le informazioni.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davraf-amuro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
