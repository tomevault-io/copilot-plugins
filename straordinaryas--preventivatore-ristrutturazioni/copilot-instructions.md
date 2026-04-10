## preventivatore-ristrutturazioni

> **Applicazione web MVP completa** per generare preventivi di ristrutturazione edilizia con prezzario base Piemonte 2025.

# Preventivatore Ristrutturazioni - Documentazione per Claude

## 📋 Panoramica Progetto

**Applicazione web MVP completa** per generare preventivi di ristrutturazione edilizia con prezzario base Piemonte 2025.

### Stack Tecnologico
- **Frontend**: Next.js 15.5.6, React 19, TypeScript, Tailwind CSS
- **Backend**: Supabase (PostgreSQL con Row Level Security)
- **Deployment**: Vercel (configurato con sottodomini Infomaniak)
- **State Management**: React hooks (useState, useEffect)

---

## 🗄️ Architettura Database

### Namespace: `ristrutturazioni_*`

Tutte le tabelle usano il prefisso `ristrutturazioni_` per isolamento logico.

#### Tabelle Principali

**1. `ristrutturazioni_progetti`**
```sql
- id (uuid, PK)
- nome (text) - Nome progetto
- mq_totali (numeric)
- piano (integer)
- ha_ascensore (boolean)
- livello_finiture (text) - 'economy' | 'standard' | 'premium'
- workflow_mode (text) - 'auto' | 'manual'
- stato (text) - 'bozza' | 'calcolato' | 'approvato'
- perc_oneri_sicurezza (numeric) - Default 0.02 (2%)
- perc_spese_generali (numeric) - Default 0.10 (10%)
- perc_utile_impresa (numeric) - Default 0.10 (10%)
- pratiche_tecniche_importo (numeric) - Default 3200
- perc_contingenze (numeric) - Default 0.07 (7%)
- perc_iva (numeric) - Default 0.10 (10%)
- progetto_originale_id (uuid) - Per versioning
- duplicato_da (text) - Descrizione origine
- created_at, updated_at
```

**2. `ristrutturazioni_computi`**
```sql
- id (uuid, PK)
- progetto_id (uuid, FK)
- lavori_base (numeric)
- oneri_sicurezza (numeric)
- spese_generali (numeric)
- utile_impresa (numeric)
- pratiche_tecniche (numeric)
- contingenze (numeric)
- imponibile (numeric)
- iva (numeric)
- totale (numeric)
- breakdown_categorie (jsonb)
- coeff_accesso (numeric)
- coeff_complessita (numeric)
- versione (integer)
- created_at
```

**3. `ristrutturazioni_categorie`**
```sql
- id (uuid, PK)
- codice (text, unique)
- nome (text)
- descrizione (text)
- ordine (integer)
- attiva (boolean)
- created_at, updated_at
```

**4. `ristrutturazioni_sottocategorie`**
```sql
- id (uuid, PK)
- id_categoria (uuid, FK)
- codice (text, unique)
- nome (text)
- descrizione (text)
- unita_misura (text) - 'mq', 'ml', 'cad', etc.
- prezzo_economy (numeric)
- prezzo_standard (numeric)
- prezzo_premium (numeric)
- applica_f_accesso (boolean)
- ordine (integer)
- attiva (boolean)
- created_at, updated_at
```

**5. `ristrutturazioni_selezioni_progetto`**
```sql
- id (uuid, PK)
- id_progetto (uuid, FK)
- id_sottocategoria (uuid, FK)
- quantita (numeric)
- prezzo_unitario_custom (numeric) - Override prezzo
- prezzo_a_corpo (numeric) - Prezzo fisso totale
- usa_prezzo_a_corpo (boolean)
- note (text)
- created_at, updated_at
```

**6. `ristrutturazioni_prezzi_custom`**
```sql
- id (uuid, PK)
- id_sottocategoria (uuid, FK, unique)
- prezzo_economy_custom (numeric)
- prezzo_standard_custom (numeric)
- prezzo_premium_custom (numeric)
- note (text)
- created_at, updated_at
```

### Migrations Eseguite

- ✅ **Migration 003**: Creazione tabelle workflow manuale (categorie, sottocategorie, selezioni)
- ✅ **Migration 004**: Aggiunta percentuali editabili a progetti
- ✅ **Migration 005**: Aggiunta tabella prezzi_custom + campo usa_prezzo_a_corpo

---

## 🏗️ Struttura File Progetto

```
preventivi-ristrutturazioni-app/
├── app/
│   ├── page.tsx                    # ✅ Dashboard - Lista progetti (228 righe)
│   ├── prezzi/
│   │   └── page.tsx               # Gestione prezzi custom globali
│   ├── admin/
│   │   └── prezzario/
│   │       └── page.tsx           # CRUD categorie/sottocategorie
│   └── preventivo/                # ✅ Refactoring 3-page completato!
│       ├── nuovo/
│       │   └── page.tsx          # ✅ Form nuovi preventivi (920 righe)
│       └── [id]/
│           └── page.tsx          # ✅ Vista/modifica progetto (1165 righe)
├── lib/
│   ├── supabase.ts               # Client Supabase + TypeScript types
│   └── pricing-engine-manual.ts  # Backend logic (17 metodi, 937 righe)
├── supabase/
│   └── migrations/
│       ├── 003_manual_workflow.sql
│       ├── 004_update_percentuali_editabili.sql
│       └── 005_prezzi_custom_e_prezzo_a_corpo.sql
├── .env.local                     # Environment variables (gitignored)
├── CHANGELOG.md                   # Storico sviluppo completo
└── CLAUDE.md                      # Questo file
```

---

## 🎯 Features Implementate (MVP 100%)

### ✅ 1. Workflow Manuale Completo
- Selezione categorie → sottocategorie → quantità
- Calcolo automatico con coefficienti (accesso, complessità)
- Formule PRD complete per riepilogo economico

### ✅ 2. Gestione Progetti
- **Salva**: Crea nuovo progetto con versioning automatico
- **Carica**: Ripristina tutti i dati (info + percentuali + selezioni)
- **Duplica**: Copia completa con nuovo nome
- **Elimina**: Cascade delete (progetto + selezioni + computi)
- **Versioning automatico**: Nome → Nome v2 → Nome v3

### ✅ 3. Prezzi Custom (3 livelli)
- **Livello 1 - Prezzario base**: Default da DB (Piemonte 2025)
- **Livello 2 - Prezzi custom globali**: `/prezzi` - Override permanenti
- **Livello 3 - Prezzi custom progetto**: Override per singolo preventivo
- **Livello 4 - Prezzo a corpo**: Prezzo fisso totale (ignora quantità)

### ✅ 4. Percentuali Economiche Editabili
Ogni progetto può personalizzare:
- Oneri sicurezza (default 2%)
- Spese generali (default 10%)
- Utile impresa (default 10%)
- Pratiche tecniche (default €3200)
- Contingenze (default 7%)
- IVA (default 10%)

### ✅ 5. Admin - Gestione Prezzario CRUD
Pagina `/admin/prezzario`:
- Crea/modifica/elimina categorie
- Crea/modifica/elimina sottocategorie
- Toggle attiva/disattiva (soft delete)
- Modali per form
- Protezioni eliminazione

### ✅ 6. Prezzo a Corpo
- Checkbox nella tabella quantità/prezzi
- Quando attivo: ignora quantità e prezzo unitario
- Campo "Prezzo totale" con stile evidenziato
- Priorità massima nel calcolo

---

## 🔧 Backend: PricingEngineManual

File: `lib/pricing-engine-manual.ts`

### Metodi Principali (17 totali)

#### Catalogo e Calcolo
```typescript
static async caricaCatalogo(): Promise<CategoriaConSottocategorie[]>
static async calcolaPreventivo(params: CalcoloParams): Promise<CalcoloManualResult>
```

#### Gestione Progetti
```typescript
static async caricaProgetti(limit?: number): Promise<Progetto[]>
static async duplicaProgetto(id_progetto: string, nuovoNome: string)
static async eliminaProgetto(id_progetto: string): Promise<{success: boolean, error?: string}>
```

#### Selezioni
```typescript
static async salvaSelezioni(id_progetto: string, voci: VoceSelezione[])
```

#### Prezzi Custom Globali
```typescript
static async caricaTuttiPrezziCustom(): Promise<PrezzoCustom[]>
static async salvaPrezzoCustomGlobale(id_sottocategoria: string, economy?, standard?, premium?)
static async eliminaPrezzoCustomGlobale(id_sottocategoria: string)
```

#### Admin Categorie
```typescript
static async creaNuovaCategoria(data: {codice, nome, descrizione, ordine})
static async modificaCategoria(id: string, data: {nome, descrizione})
static async eliminaCategoria(id: string)
static async toggleCategoria(id: string)
```

#### Admin Sottocategorie
```typescript
static async creaNuovaSottocategoria(data: {id_categoria, codice, nome, unita_misura, prezzi, ...})
static async modificaSottocategoria(id: string, data: {...})
static async eliminaSottocategoria(id: string)
static async toggleSottocategoria(id: string)
```

### Formule Calcolo (PRD)

```typescript
// 1. Somma lavori base
lavori_base = Σ(quantità × prezzo_unitario_finale)

// 2. Applica coefficiente accesso (se sottocategoria.applica_f_accesso)
if (piano > 0 && !ha_ascensore) {
  coeff_accesso = 1 + (0.03 × piano)  // +3% per piano
}
lavori_base_con_accesso = lavori_base × coeff_accesso

// 3. Calcola spese
oneri_sicurezza = lavori_base_con_accesso × perc_oneri_sicurezza
spese_generali = lavori_base_con_accesso × perc_spese_generali
utile_impresa = lavori_base_con_accesso × perc_utile_impresa
pratiche_tecniche = importo_fisso
contingenze = (lavori + oneri + spese + utile + pratiche) × perc_contingenze

// 4. Imponibile e IVA
imponibile = lavori + oneri + spese + utile + pratiche + contingenze
iva = imponibile × perc_iva
totale = imponibile + iva
```

### Priorità Prezzi (4 livelli)

```typescript
if (voce.usa_prezzo_a_corpo) {
  prezzo_finale = voce.prezzo_a_corpo  // Livello 1 (max priorità)
} else if (voce.prezzo_unitario_custom) {
  prezzo_finale = voce.prezzo_unitario_custom × quantità  // Livello 2
} else if (prezzoCustomGlobale[livello]) {
  prezzo_finale = prezzoCustomGlobale[livello] × quantità  // Livello 3
} else {
  prezzo_finale = sottocategoria[`prezzo_${livello}`] × quantità  // Livello 4 (base)
}
```

---

## 🎨 Frontend: app/page.tsx

### State Management (React Hooks)

```typescript
// Catalog
const [catalogo, setCatalogo] = useState<CategoriaConSottocategorie[]>([])
const [loadingCatalogo, setLoadingCatalogo] = useState(true)

// Project input
const [nomeProgetto, setNomeProgetto] = useState('')
const [mq, setMq] = useState(90)
const [piano, setPiano] = useState(4)
const [ascensore, setAscensore] = useState(false)
const [livelloFiniture, setLivelloFiniture] = useState<'economy' | 'standard' | 'premium'>('standard')

// Percentuali editabili
const [percOneriSicurezza, setPercOneriSicurezza] = useState(2)
const [percSpeseGenerali, setPercSpeseGenerali] = useState(10)
const [percUtileImpresa, setPercUtileImpresa] = useState(10)
const [importoPraticheTecniche, setImportoPraticheTecniche] = useState(3200)
const [percContingenze, setPercContingenze] = useState(7)
const [percIVA, setPercIVA] = useState(10)

// Selezioni
const [vociSelezionate, setVociSelezionate] = useState<VoceSelezione[]>([])

// Calcolo
const [risultato, setRisultato] = useState<CalcoloManualResult | null>(null)
const [calculating, setCalculating] = useState(false)
const [showDettaglio, setShowDettaglio] = useState(false)

// Progetti salvati
const [progettoSalvato, setProgettoSalvato] = useState<Progetto | null>(null)
const [progettiSalvati, setProgettiSalvati] = useState<Progetto[]>([])
const [loadingProgetti, setLoadingProgetti] = useState(false)
const [showProgettiList, setShowProgettiList] = useState(true)
```

### Funzioni Principali

```typescript
// Caricamento dati
caricaCatalogo(): void
caricaListaProgetti(): void

// Gestione selezioni
selezionaSottocategoria(sottocat): void
rimuoviSelezione(id): void
aggiornaQuantita(id, quantita): void
aggiornaPrezzoCustom(id, prezzo): void
togglePrezzoACorpo(id, usaACorpo): void
aggiornaPrezzoACorpo(id, prezzo): void

// Gestione progetti
calcolaPreventivo(): void  // Calcola + Salva + Versioning
caricaProgetto(progetto): void
duplicaProgetto(progetto): void
eliminaProgetto(progetto): void
nuovoProgetto(): void  // Reset form
```

### Sistema Versioning

Quando si salva un progetto:

1. **Nome nuovo** → Crea progetto con quel nome
2. **Nome esistente** → Crea automaticamente versione:
   - `Appartamento Via Roma` → `Appartamento Via Roma v2`
   - `Appartamento Via Roma v2` → `Appartamento Via Roma v3`
3. **Alert informativo** quando viene creata nuova versione
4. **Badge "Versione"** arancione nella lista progetti
5. **Campo `duplicato_da`** traccia l'origine: "Aggiornamento di: Nome Originale"

**Vantaggi:**
- ✅ Storico completo modifiche
- ✅ Zero rischio sovrascrittura
- ✅ Data creazione sempre aggiornata
- ✅ Confronto versioni possibile

---

## 🚀 Deployment su Vercel con Infomaniak

### ✅ STATUS: DEPLOYMENT COMPLETATO

**URL Produzione:**
```
https://preventivatore-ristrutturazioni.geko-it.com
```

**Repository GitHub:**
```
https://github.com/StraordinaryAS/Preventivatore_Ristrutturazioni
```

**Vercel Project:**
```
preventivatore-ristrutturazioni
```

### Setup Completato

#### ✅ 1. GitHub Repository
- Repository creato e collegato a Vercel
- Branch: `main`
- Auto-deploy attivo su push

#### ✅ 2. Vercel Configuration
- Framework: Next.js 15.5.6 (auto-detect)
- Root Directory: `.` (repository is the project folder)
- Environment Variables configurate:
  ```
  NEXT_PUBLIC_SUPABASE_URL = https://sngyhrzlblokthugamib.supabase.co
  NEXT_PUBLIC_SUPABASE_ANON_KEY = [configurato]
  ```
- Build Command: `npm run build`
- Output Directory: `.next`

#### ✅ 3. DNS Configuration (Infomaniak)
**Sottodominio:** `preventivatore-ristrutturazioni.geko-it.com`

Record DNS configurato:
```
Tipo: CNAME
Nome: preventivatore-ristrutturazioni
Valore: cname.vercel-dns.com
TTL: 3600
```

**Note:** Usato trattino `-` invece di underscore `_` per compatibilità DNS.

#### ✅ 4. Supabase Authentication URLs
Dashboard Supabase → Authentication → URL Configuration:
```
Site URL: https://preventivatore-ristrutturazioni.geko-it.com
Redirect URLs:
  - https://preventivatore-ristrutturazioni.geko-it.com/*
  - https://preventivatore-ristrutturazioni.vercel.app/*
  - http://localhost:3000/*
```

### Deployment Workflow

Ogni push su branch `main` triggera automaticamente:
1. Build su Vercel
2. TypeScript type checking
3. Deploy su produzione (se build success)

```bash
# Workflow standard
git add .
git commit -m "feat: descrizione"
git push origin main
# → Vercel auto-deploy in ~2 minuti
```

### Build Verificato

```
✓ Compiled successfully
✓ Linting and checking validity of types
✓ Generating static pages (7/7)
✓ Finalizing page optimization

Route (app)                    Size  First Load JS
┌ ○ /                       1.89 kB       162 kB
├ ○ /_not-found              993 B        103 kB
├ ○ /admin/prezzario        3.22 kB       163 kB
├ ƒ /preventivo/[id]        5.87 kB       166 kB
├ ○ /preventivo/nuovo       4.7 kB        165 kB
└ ○ /prezzi                 2.72 kB       163 kB

○  (Static)   prerendered as static content
ƒ  (Dynamic)  server-rendered on demand
```

### Vantaggi Sottodomini
- ✅ Ogni app completamente indipendente
- ✅ Deploy separati (non si influenzano)
- ✅ Facile aggiungere nuove app in futuro
- ✅ Performance migliori
- ✅ SSL automatico (Let's Encrypt)

---

## ✅ Refactoring Architettura 3-Page COMPLETATO

### Obiettivo Raggiunto: Struttura 3 Pagine

**Motivazione:** Separare dashboard (lista progetti) da editor (crea/modifica preventivo).

### Struttura Implementata

```
app/
├── page.tsx                    # ✅ Dashboard - Lista progetti (228 righe)
├── preventivo/
│   ├── nuovo/
│   │   └── page.tsx           # ✅ Form nuovo preventivo (920 righe)
│   └── [id]/
│       └── page.tsx           # ✅ Visualizza E modifica (1165 righe)
```

### Workflow Utente Implementato

**Dashboard (`/`):**
- ✅ Lista tutti i progetti salvati con card
- ✅ Info: Nome, MQ, Piano, Ascensore, Finiture, Data creazione
- ✅ Badge "Versione" per progetti v2, v3, etc.
- ✅ Campo "Aggiornamento di:" per progetti duplicati
- ✅ Bottoni: `+ Nuovo Preventivo`, `Visualizza`, `Duplica`, `Elimina`
- ✅ Links navigazione: `Prezzi Custom`, `Admin Prezzario`

**Preventivo Nuovo (`/preventivo/nuovo`):**
- ✅ Form completo per nuovo preventivo
- ✅ Dati base: Nome, MQ, Piano, Ascensore, Finiture
- ✅ Percentuali economiche editabili
- ✅ Selezione categorie → sottocategorie
- ✅ Tabella quantità con prezzi custom e prezzo a corpo
- ✅ Calcolo real-time con breakdown categorie
- ✅ Al salvataggio → redirect a `/preventivo/[id]` in modalità view
- ✅ Sistema versioning automatico (v2, v3, v4...)

**Preventivo Esistente (`/preventivo/[id]`):**

✅ **2 modalità implementate:**

**1. Modalità Vista (default - `mode=view`):**
- ✅ Read-only con tutti i dati visualizzati
- ✅ Tabella voci selezionate con prezzi finali
- ✅ Breakdown categorie
- ✅ Riepilogo economico completo
- ✅ Bottoni: `Modifica`, `Duplica`, `Elimina`, `← Torna alla Dashboard`

**2. Modalità Modifica (`mode=edit`):**
- ✅ Form editabile completo (identico a "nuovo")
- ✅ Pre-popolato con tutti i dati esistenti
- ✅ Al salvataggio: sistema versioning automatico
  - Se nome uguale a esistente → crea v2/v3/etc
  - Se nome nuovo → crea nuovo progetto
- ✅ Alert informativo quando viene creata nuova versione
- ✅ Redirect a `/preventivo/[nuovo_id]?mode=view`

### Breaking Changes - Next.js 15

⚠️ **IMPORTANTE:** Next.js 15 ha introdotto breaking change con `params`:

```typescript
// ❌ VECCHIO (non funziona più)
export default function Page({ params }: { params: { id: string } }) {
  useEffect(() => {
    doSomething(params.id)
  }, [params.id])
}

// ✅ NUOVO (Next.js 15+)
import { use } from 'react'

export default function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = use(params)  // Unwrap Promise

  useEffect(() => {
    doSomething(id)
  }, [id])  // usa id, non params.id
}
```

**Errore console se non corretto:**
```
A param property was accessed directly with `params.id`.
`params` is now a Promise and should be unwrapped with `React.use()`
```

### Tasks Completati

- [x] Creare struttura cartelle `app/preventivo`
- [x] Trasformare `app/page.tsx` in dashboard lista progetti
- [x] Creare `app/preventivo/nuovo/page.tsx`
- [x] Creare `app/preventivo/[id]/page.tsx` con logica view/edit
- [x] Implementare sistema versioning automatico (v2, v3, v4...)
- [x] Fix Next.js 15 params Promise con `React.use()`
- [x] Fix 4 errori TypeScript per deployment
- [x] Merge branch feature-refactoring → main
- [x] Cleanup file backup obsoleti
- [x] Deploy su Vercel con successo

---

## 📝 Note Importanti

### Environment Variables (.env.local)
```bash
NEXT_PUBLIC_SUPABASE_URL=https://[project-id].supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=[anon-key]
```

**⚠️ NON committare .env.local su GitHub!**

### Row Level Security (RLS)

Attualmente **RLS è disabilitato** su tutte le tabelle per semplicità MVP.

Per produzione con autenticazione:
```sql
ALTER TABLE ristrutturazioni_progetti ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own projects"
ON ristrutturazioni_progetti FOR SELECT
USING (auth.uid() = user_id);
```

### TypeScript Types

Tutti i tipi sono definiti in `lib/supabase.ts`:
- `Progetto`
- `Computo`
- `Categoria`
- `Sottocategoria`
- `SelezioneProgetto`
- `PrezzoCustom`

### Commit Convention

Usa conventional commits:
```
feat: nuova funzionalità
fix: bug fix
refactor: refactoring
docs: documentazione
chore: manutenzione
```

---

## 🐛 Known Issues / Limitazioni

1. **No autenticazione utente** - MVP single-tenant
2. **No export PDF/Excel** - Da implementare
3. **No stampa professionale** - Da implementare (prossima feature)
4. **No multilingua** - Solo italiano
5. **No responsive mobile** - Ottimizzato per desktop
6. **Prezzario** - ~150 voci Piemonte 2025, 14 categorie

---

## 📚 Risorse Utili

### Documentazione
- [Next.js 15](https://nextjs.org/docs)
- [Supabase](https://supabase.com/docs)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Vercel](https://vercel.com/docs)

### Repository
- **GitHub:** https://github.com/StraordinaryAS/Preventivatore_Ristrutturazioni
- **Vercel:** https://vercel.com/straordinaryass-projects/preventivatore-ristrutturazioni
- **Supabase:** https://sngyhrzlblokthugamib.supabase.co
- **Produzione:** https://preventivatore-ristrutturazioni.geko-it.com

---

## 🎯 Quick Start per Nuova Chat

1. Leggi questo file per context completo
2. Controlla `CHANGELOG.md` per storico dettagliato
3. Esplora `lib/pricing-engine-manual.ts` per logica backend (937 righe)
4. Vedi pagine refactored:
   - `app/page.tsx` - Dashboard (228 righe)
   - `app/preventivo/nuovo/page.tsx` - Nuovo preventivo (920 righe)
   - `app/preventivo/[id]/page.tsx` - Dettaglio/Edit (1165 righe)
5. Migrations in `supabase/migrations/` per schema DB

**Comando dev:**
```bash
cd "c:\Development\GEKO\App preventivo Ristrutturazioni\preventivi-ristrutturazioni-app"
npm run dev
```

**URLs:**
- Locale: http://localhost:3000
- Produzione: https://preventivatore-ristrutturazioni.geko-it.com

---

## 🎯 Prossime Features da Implementare

1. **Stampa Preventivo Professionale** 🖨️
   - Pagina `/preventivo/[id]/stampa` con layout print-friendly
   - Note personalizzate per ogni voce
   - Pagina iniziale con intro cliente
   - Blocchi testo personalizzabili
   - Logo GEKO + dati cliente + footer azienda
   - Export PDF

2. **Export Excel/CSV**
   - Download foglio calcolo con breakdown completo

3. **Autenticazione utenti**
   - Multi-tenant con Supabase Auth
   - Row Level Security

4. **Mobile responsive**
   - Layout ottimizzato per tablet/smartphone

---

**Versione documento:** 2025-12-01
**Stato progetto:** ✅ MVP 100% completo e deployato in produzione
**Ultima sessione:** Deployment Vercel completato + Fix TypeScript errors
**Prossima feature:** Sistema stampa preventivo professionale

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StraordinaryAS)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/StraordinaryAS)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
