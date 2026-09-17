## progettostudio

> Valgono per qualsiasi assistente o modello linguistico che lavori in questa

# Istruzioni per gli agenti

Valgono per qualsiasi assistente o modello linguistico che lavori in questa
cartella, e per chiunque preferisca leggere una pagina sola.

## Che cos'è questo progetto

Una biblioteca di studio interattiva in italiano sugli LLM. Ogni "libro" è un
file `.dc.html` autonomo che si apre nel browser con un doppio clic: testo,
formule, piccole demo. Nessuna build. La homepage è `Biblioteca.dc.html`.

## La regola d'oro

**Prima di creare o modificare un file `.dc.html`, apri due documenti:**

| Documento | Risponde a |
|---|---|
| [AgentFE.md](docs/AgentFE.md) | **Com'è fatta** una pagina: classi, colori, icone, la procedura in tre passi |
| [AgentAutore.md](docs/AgentAutore.md) | **Come si scrive** un libro: i sette principi didattici, il procedimento, la verifica |

Vale nei tre casi: libro nuovo chiesto a voce, libro nuovo ricavato da un
documento fornito, modifica o ampliamento di un libro esistente. Non serve che
la richiesta lo ricordi — è il modo di lavorare predefinito del progetto.

## Prima di cominciare, guarda chi altro sta lavorando

Su questo progetto lavorano più sessioni in parallelo, e **`git branch -a`
mostra solo ciò che il clone ha già scaricato** — in una sessione appena creata
può essere una fotografia parziale. Le due domande da fare al server, non al
clone:

```bash
git ls-remote --heads origin     # tutti i rami che esistono davvero
```

più l'elenco delle **pull request aperte**. Un libro scritto altrove, o una
homepage già ridisegnata, cambiano completamente il lavoro da fare: scoprirlo
dopo costa il doppio, e rischia di rifare da capo cose già fatte.

## Il ciclo di lavoro

1. Stabilisci da dove parte il lettore. Se non è deducibile, chiedilo.
2. **Cerca le fonti online prima di scrivere.** Numeri, date e attribuzioni si
   verificano; ciò che non è verificato si riscrive al ribasso. Su questi temi
   cambia molto in fretta.
3. Scegli le analogie portanti, una per concetto difficile.
4. Disegna le tappe: da quale noto parte ognuna, cosa aggiunge, quale concetto
   precedente riapre.
5. Scrivi partendo da `_TEMPLATE.dc.html`.
6. Registra il modulo in `wiki/wiki-index.js` — senza quel passo la pagina
   esiste ma non compare nella Biblioteca. Ogni concetto dichiara la sua
   `tappa` (`"c4"`): è l'àncora con cui il grafo apre il libro al punto giusto.
7. **Esegui i due verificatori.** Zero errori, sempre.
   `node wiki/verifica.js` legge il sorgente: struttura del libro, catena che
   fa esistere l'indice laterale, corrispondenza concetti ↔ tappe, i due
   divieti qui sotto. `node strumenti/verifica-pagine.js` apre un browser:
   testo sovrapposto, sfondamenti, binding visibili, demo morte.
8. Verifica il resto a mano: la checklist di contenuto in `docs/AgentAutore.md`,
   quella di forma in `docs/AgentFE.md`.

## I due divieti che contano

- **Mai `style="..."`, mai colori scritti a mano.** Tutto passa dalle classi
  `w-` e dai token di `wiki/wiki.css`. Unica eccezione: dentro una demo, dove
  lo stile calcolato è il punto stesso della demo. Le altre tre regole di forma
  stanno in `docs/AgentFE.md`. Non è un principio morale: è controllato da
  `node wiki/verifica.js`, e fallisce.
- **Mai un dato non verificato.** Nessun numero, primato, data o attribuzione
  che non regga a un controllo. È l'unica regola di questo progetto senza
  eccezioni: vedi il principio 4 in `docs/AgentAutore.md`.

## Dove vanno le cose

| Cosa | Dove |
|---|---|
| I libri (`.dc.html`) e `index.html` | in **radice** — sono gli URL pubblici di GitHub Pages, non spostarli |
| Il connettore condiviso | `wiki/` |
| La documentazione | `docs/` — tranne `README.md`, `AGENTS.md`, `CLAUDE.md`, `INDEX.md`, che stanno in radice per convenzione |
| Le decisioni di progettazione | `docs/decisioni/`, un file per decisione, `AAAA-MM-GG-argomento-design.md` |

## Non toccare

`support.js` — è il runtime `dc-runtime` compilato altrove.

---

Mappa completa della documentazione: [INDEX.md](INDEX.md).

---
> Source: [TizianoCarpentieri/ProgettoSTUDIO](https://github.com/TizianoCarpentieri/ProgettoSTUDIO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
