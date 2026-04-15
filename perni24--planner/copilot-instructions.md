## planner

> Questo file contiene le linee guida fondamentali per lo sviluppo del progetto **Planner**. Queste istruzioni hanno la precedenza su qualsiasi altro workflow predefinito.

# Project Mandates (GEMINI.md)

Questo file contiene le linee guida fondamentali per lo sviluppo del progetto **Planner**. Queste istruzioni hanno la precedenza su qualsiasi altro workflow predefinito.

## 🎯 Obiettivi del Progetto
- Creare un'applicazione di pianificazione (Planner) moderna, veloce e reattiva.
- Mantenere una netta separazione tra Frontend (React) e Backend (Python/Starlette).

## 🎨 Standard Frontend
- **Framework**: React 19 con Vite.
- **Styling**: Utilizzare esclusivamente **Tailwind CSS 4**. Evitare CSS inline o moduli CSS separati a meno di estrema necessità.
- **Routing**: Utilizzare **React Router 7**. Preferire l'approccio con `BrowserRouter` o `createBrowserRouter` in `App.jsx`.
- **Componenti**: I nomi dei file e delle funzioni devono usare il **PascalCase** (es. `Header.jsx`).
- **Architettura**:
    - `src/components/`: Componenti UI riutilizzabili.
    - `src/pages/`: Componenti che rappresentano intere viste/pagine.
    - `src/types/`: Definizioni TypeScript.

## ⚙️ Standard Backend
- **Linguaggio**: Python 3.14+.
- **Framework**: Starlette (leggero e performante).
- **Database**: SQLite 3 (gestito tramite il modulo standard `sqlite3`).
- **Ambiente**: Utilizzare sempre il virtual environment (`.venv`).

## 🛠️ Regole di Sviluppo
- **Documentazione**: Ogni nuova funzionalità significativa deve essere riflessa in `doc/tecnologie.md`.
- **Sicurezza**: Mai includere segreti o chiavi API nel codice sorgente. Utilizzare file `.env` (non tracciati da Git).
- **Lingua**: Commenti e documentazione tecnica in italiano. Nomi di variabili e funzioni in inglese (standard di settore).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/perni24)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/perni24)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
