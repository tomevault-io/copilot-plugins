## frametech-sl

> > ℹ️ **ATTENZIONE: Tutte le procedure tecniche dettagliate, policy CI/CD, workflow Vercel/Next.js/OpenAI, fix errori, troubleshooting e script sono documentati nella guida [guida-vercel.md](./guida-vercel.md) nella root. Consultare SEMPRE quella guida prima di modificare script, policy o proporre fix automatici.**

# AGENTS.md

> ℹ️ **ATTENZIONE: Tutte le procedure tecniche dettagliate, policy CI/CD, workflow Vercel/Next.js/OpenAI, fix errori, troubleshooting e script sono documentati nella guida [guida-vercel.md](./guida-vercel.md) nella root. Consultare SEMPRE quella guida prima di modificare script, policy o proporre fix automatici.**


## 🚀 Build, Test, Lint, Deploy

- **Install dependencies:**  
  - Se usi pnpm: `pnpm install`  
  - Se usi npm: `npm install`
- **Build:**  
  - `pnpm build --filter <project_name>`  
  - Oppure: `npm run build`
- **Test:**  
  - `pnpm turbo run test --filter <project_name>`  
  - Oppure: `npm test`
  - Esegui solo uno specifico: `pnpm vitest run -t "<test name>"`
- **Lint:**  
  - `pnpm lint --filter <project_name>`  
  - Oppure: `npm run lint`
- **Start dev:**  
  - `pnpm dev --filter <project_name>`  
  - Oppure: `npm run dev`
- **Deploy:**  
  - `vercel --prod` (se necessario)
- **Watcher AI/Debug Vercel:**  
  - Avvia `node ai-deploy-watcher.js` per monitoraggio errori deploy/CI con analisi automatica AI (vedi sezione sotto).

---

## 🧠 AI, Codex, e Script Watcher

- **Script di monitoraggio errori deploy:**  
  - Lo script `ai-deploy-watcher.js` controlla gli errori di deploy Vercel, legge i log e li invia ad OpenAI per ricevere una spiegazione e un fix AI.
  - Viene lanciato automaticamente dallo “Script di configurazione” del workspace.
  - **Tutti gli errori di deploy vanno letti nei log di `ai-deploy-watcher.log` prima di aprire una Pull Request.**
- **Policy AI/Codex:**  
  - Codex può proporre fix automatici SOLO se la build e i test sono verdi (no merge con build rossa).
  - Se la build fallisce, il watcher fornisce log dettagliato e suggerisce il fix AI.
  - Aggiorna AGENTS.md se cambia lo script watcher, la policy CI o la struttura del progetto.

---

## 🧪 Dev Environment & Toolchain

- Per nuovi pacchetti React+Vite:  
  `pnpm create vite@latest <project_name> -- --template react-ts`
- Usa sempre la versione Node.js >= 20.x
- Conferma che il campo `"name"` sia corretto in ogni `package.json` di sub-package.
- Usa `pnpm dlx turbo run where <project_name>` per individuare la directory.

---

## 📦 Workspace Policy (Monorepo)
- Non lasciare pacchetti inutilizzati o orphan.
- Installa solo le dipendenze necessarie su ogni sotto-pacchetto.
- Aggiorna la documentazione ogni volta che cambia la struttura.

---

## 📝 PR & Commit Instructions

- **Titolo PR:** `[<project_name>] Breve descrizione`
- **Commit message:** chiaro, descrittivo, riferito a issue/task.
- **PR message:** deve includere:
  - Sommario delle modifiche
  - Output di test (“Testing done”) e, se utile, log generato dallo script AI watcher
  - Issue reference (se c’è)
- **Prima del merge:**  
  - Assicurati che TUTTI i test/lint/build siano verdi  
  - Verifica che il watcher AI non abbia segnalato errori Vercel non risolti

---

## 🔐 Security & Best Practice

- Non committare mai `.env`, chiavi, token o segreti (usa GitHub/Vercel Secrets).
- Le variabili d’ambiente richieste (`OPENAI_API_KEY`, `VERCEL_TOKEN`, `PROJECT_ID`, ecc.) vanno impostate nelle rispettive UI cloud, **mai in chiaro nei file**.
- Aggiorna AGENTS.md e README.md se cambi toolchain, policy o script.

---

## 📚 Documentazione & Manutenzione

- Mantieni README.md e AGENTS.md sempre allineati con i comandi e policy correnti.
- Aggiorna la documentazione dopo ogni refactoring, cambio toolchain, aggiornamento CI/CD, o modifica importante.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rootkalilinuxok) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
