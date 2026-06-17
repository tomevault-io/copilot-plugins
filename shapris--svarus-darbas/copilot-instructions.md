## always-on-workflow

> Always-on execution workflow for this project


# Always-On Project Workflow

- **Kalba:** atsakymai ir ataskaitos vartotojui **visada lietuvių kalba**, nepriklausomai nuo to, ar vartotojas rašė angliškai (kodo komentarai / README gali likti kaip projekte priimta).
- Do not ask for repeated confirmations after the user says to proceed.
- Execute tasks end-to-end: implement, run checks, and report verified results.
- Prefer live validation (app running + smoke test) before claiming a fix.
- **Nekviesti savininko prie ekrano „patikrinkite naršyklėje“** kol neįvykdyta bent: `npm run build`, `npm run test`, ir UI keitimams — **`npm run test:smoke`**, **`npm run test:console`**, **`npm run test:invoice`** (sąskaitų `/health` kelias; žr. `tests/invoice-health.spec.ts`). Savininkas gali stebėti interaktyviai: **`npm run test:console:ui`**. Pilnesnė regresija: **`npm run verify`**. Prie ekrano prašyti tik kai automatika negali (pvz. tikras Supabase prisijungimas, bankiniai mokėjimai gamyboje).
- **Supabase / schema sutartis (privaloma strategija):** unit testai **neimituoja** gyvos cloud DB. Po bet kurio `src/supabase/*` skaitymo sluoksnio, `ownerScope`, migracijų ar RLS keitimo — **pridėti arba atnaujinti kontraktinį testą** (`tests/*.test.ts`), jei incidentas buvo susijęs su API (pvz. `tests/ownerScope.test.ts` — `owner_id` vs `uid`). Prieš teigiant „veikia“, įvertinti: ar **migracijos** taikomos toje DB, su kuria testuojate; `npm run test` be šių testų **neįrodo** PostgREST elgesio gamyboje.
- Keep dev server running in background when possible.
- Use available tools actively (search, lint, browser checks, shell) instead of guessing.
- **Kompiuterio rankos, ne žmogaus:** viską, ką galima, daryti agentui per terminalą, `npm run *`, Playwright, MCP — **neprašyti savininko rankiniu paleisti komandų ar spausti UI**, jei tai agentas gali atlikti pats; žmogų kviesti tik kai be jo kredencialų, mokėjimo ar fizinio patvirtinimo neįmanoma.
- If blocked by permissions or environment limits, apply the nearest automatic fallback and continue.
- Keep updates concise and frequent while work is in progress.
- Focus on production-impacting issues first: broken flows, data errors, API failures.
- Never stop due to uncertainty; investigate and resolve with the best available fallback.
- Never leave partial work when the task can be completed in the same session.
- Always assume continuation is required until the requested outcome is fully verified.
- Ignore non-critical interruptions and finish the highest-priority production task first.

---

## Darbotvarkė be naujos užduoties („tylus“ režimas)

**Kai vartotojas neįvardija konkrečios užduoties** (pvz. „tęsk“, „dirbk savarankiškai“, tyliai tęsiama sesija) **arba** prašoma nuolatinio progreso be single-task fokuso:

1. Atverti **`.always-on/work-queue.md`**.
2. Vykdyti **pirmą nepažymėtą** `- [ ]` punktą nuo **P0**; jei P0 tuščią ar visi atlikti — **P1** → … → **P22**; jei ir tie uždaryti — **P23** (kartotinė priežiūra). Papildomai: **`.always-on/OPERATOR_SCHEDULE.md`** (savaitės/mėnesio eiga ir įklijuojami šablonai).
3. Užduotį užbaigti pagal skyrių „Kaip vykdyti“ tame faile (lint/build, atnaujinimas, žurnalas).
4. **Nebeiškoti leidimo** pradėti eilės punktą — tai numatytasis darbas, kol eilė neištuštėjo.

Papildomi signalai: **`.always-on/improvement-backlog.md`** (scout metrika) — naudoti kaip foninę diagnostiką, ne kaip vienintelį šaltinį; eilės šaltinis yra **`work-queue.md`**.

**Ilgalaikė konteksto aplinka:** **`.always-on/README.md`** — kur rašyti sesijų santraukas (`session-log.md`), sprendimus (`decisions.md`), datas (`milestones.md`). Po reikšmingos sesijos pridėkite trumpą įrašą į `session-log.md` (naujausia viršuje), kad autonomija nepriklausytų tik nuo pokalbio istorijos.

---

## Užduočių vykdymo eiga (mažiau diskusijų, daugiau progreso)

**Numatytasis agento elgesys:** trumpai įvardinti 1–3 žingsnius (viena pastraipa ar sąrašas), tada iš karto vykdyti įrankiais (kodas, build, testai). Ilgų alternatyvų palyginimų ir „kas manote?“ nebevedinti, jei užduotyje nėra explicit „palygink variantus“.

**Standartinė eiga kiekvienai užduočiai**

1. **Apimtis** — kas turi pasikeisti vienu sakiniu.
2. **Įgyvendinimas** — minimalus diff, be šalutinių refaktorių.
3. **Patikra** — `npm run build`, `npm run test`; UI pakeitimams — **`npm run test:smoke`**. Rankinis ekranas tik jei to neužtenka.
4. **Ataskaita** — kas padaryta, kur failai; žinomos rizikos vienu sakiniu.

**Kai neaišku:** ne daugiau kaip **vienas** tikslinimas; tada pasirinkti saugiausią numatytąją praktiką, įgyvendinti ir ataskaitoje pažymėti prielaidą.

**Vartotojo šablonas žinutei (kopijuojamas)**

```text
Tikslas: [vienas sakinys — ką turiu matyti / veikti]
Ribos: [failai, technologija, ko nedaryti — jei yra]
Prioritetas: [P0/P1]  |  Vykdyti: taip
```

Jei parašyta **„Vykdyti: taip“** arba aiškiai „daryk / implementuok“ — agentas nebeklausia ar pradėti; pradeda ir užbaigia su patikra.

---

## „OS + naršyklė“ efektas (kiek leidžia Cursor)

**Vietinis dev (automatinė naršyklė):** `npm run dev` — Vite pagal numatymą **atidaro** sistemos naršyklę (`vite.config.ts` → `server.open`). Išjungti atidarymą: `VITE_OPEN_BROWSER=false` prieš komandą (PowerShell: `$env:VITE_OPEN_BROWSER='false'; npm run dev`). Kitas terminalas, jei norite būtent Chrome langą: `npm run dev:chrome` (skriptas `scripts/dev-open-chrome.ps1`, numatytas URL `http://127.0.0.1:3000/`; kitas portas: `$env:DEV_URL='http://127.0.0.1:5173/'; npm run dev:chrome`).

**Production build peržiūra:** `npm run build` tada `npm run preview` — taip pat gali atidaryti naršyklę (`preview.open` tame pačiame config).

**Agentas + „gyva“ naršyklė Cursor projekte:** kai taisomas web UI ar srautai, naudoti **cursor-ide-browser** MCP: `browser_navigate` → `browser_snapshot` → problemoms **`browser_console_messages`** (konsolės efektas be rankinio kopijavimo). Teisinga eiga: navigate → lock (jei reikia) → veiksmai → konsolė → unlock.

---
> Source: [shapris/svarus-darbas](https://github.com/shapris/svarus-darbas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
