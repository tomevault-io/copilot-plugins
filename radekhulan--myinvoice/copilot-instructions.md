## myinvoice

> Pokyny pro práci s tímto repozitářem (Claude Code, Codex, Cursor, Copilot a další).

# AGENTS.md — pravidla pro AI agenty a přispěvatele

Pokyny pro práci s tímto repozitářem (Claude Code, Codex, Cursor, Copilot a další).
Platí pro celý repozitář. Obecný popis projektu je v [README.md](README.md),
vývojářská spec v [`source/`](source/00-README.md).

## O projektu

MyInvoice.cz — český self-hosted fakturační a účetní systém (vystavené + přijaté
faktury, multi-supplier, DPH/KH/SH výkazy, EPO XML, CRM, REST API).
Backend PHP 8.5 + Slim 4, frontend Vue 3 + TypeScript + Vite + Tailwind,
databáze MariaDB 10.6+ (doporučeno 11.x).

## Layout repozitáře

- `api/` — PHP backend (Slim, autowired actions, services, repositories); `api/bin/` = CLI skripty, `api/tests/` = PHPUnit
- `web/` — Vue 3 + TS frontend; zdrojáky ve `web/src/`, lokalizace ve `web/src/i18n/`
- `dist/` — produkční build frontendu (commitovaný — uživatelé testují přes něj)
- `db/migrations/` — SQL migrace (číslované, idempotentní)
- `manual/` — uživatelský manuál (Markdown, česky); `manual/generated/` = vyrenderované HTML
- `source/` — vývojářská spec a plány
- `tools/` — pomocné skripty (generování manuálu, převody obrázků)
- `cmd/` — cron/deploy wrappery (`.sh` + `.cmd`/`.ps1`)

## Příkazy

```bash
# Frontend — build (NUTNÉ po každé změně web/src, dist/ se commituje)
cd web && pnpm build            # = vue-tsc --noEmit && vite build (npm run build funguje též)
cd web && pnpm type-check       # jen typová kontrola

# PHP testy (PHPUnit 13)
cd api && php vendor/bin/phpunit                  # vše
cd api && php vendor/bin/phpunit --filter Xyz     # podmnožina

# Migrace — VŽDY přes migrate.php, NIKDY mysql klientem přímo
php api/bin/migrate.php
php api/bin/migrate.php --status

# Manuál — regenerovat po každé změně manual/*.md
php tools/generateManualHtml.php
php tools/exportManualToPdf.php
```

## Tvrdá pravidla

### Migrace
- Nová migrace = nový číslovaný soubor v `db/migrations/`, spouští se **výhradně** přes `php api/bin/migrate.php`.
- Každá migrace musí být **idempotentní** (opakovatelně spustitelná): používej nativní MariaDB `IF [NOT] EXISTS` (`ADD COLUMN IF NOT EXISTS`, `CREATE TABLE IF NOT EXISTS`, …), ne PREPARE/EXECUTE triky.
- Cílová DB je MariaDB 10.6+: v SQL preferuj **window functions a CTE** před vnořenými subselecty; nepoužívej `SQL_CALC_FOUND_ROWS`.

### i18n
- Veškeré nové UI texty přes `t()` z vue-i18n — **nikdy** natvrdo česky/anglicky v šablonách. Vždy doplň **obě** locale (`web/src/i18n/cs.json` i `en.json`).
- Pole/seznamy překladů přes `tm()` + `rt()` — `t()` pole stringifikuje.
- Literální `{` `}` v textu zprávy escapuj jako `{'{token}'}` — jinak to vue-i18n bere jako interpolaci a render tiše spadne.

### OpenAPI sync
- Při **jakékoli** změně veřejného API (nová route, změna serializace, nový/změněný sloupec promítnutý do JSON, nové query/body pole) ihned aktualizuj `api/openapi.yaml` — jak `paths` (`/api/v1/*`), tak `components/schemas`.
- Po editaci ověř: YAML se parsuje, žádné duplicitní klíče (PyYAML je tiše přepíše — použij striktní loader), žádné dangling `$ref`.
- Veřejné API je kurátorovaný read-only subset; mutace číselníků a interní plumbing se nedokumentují.

### DPH a daňová správnost
- Veškerá evidence DPH jde přes `VatLedgerService` — nikdy neobcházet vlastním SQL.
- Výkazy a rekapitulace sumují **řádky** (`invoice_items` / per-řádkové totály), ne hlavičku dokladu.
- Při zásahu do daní/DPH proaktivně ověř daňovou správnost (zařazení do správného období), ne jen „napojení na existující kód". Kontroluj **symetrii** filtrů: obě strany evidence proti všem typům dokladů (`invoice_type` vs `document_kind`); proforma = záloha na vstupu.
- Každá nová cesta, která tvoří doklad z jiného dokladu (proforma → faktura, dobropis, kopie, recurring), musí přenést `prices_include_vat` — jinak se brutto cena chybně přepočítá jako netto.
- Agregace nákladů z `purchase_invoices` musí vyřadit spárované/zaplacené zálohové doklady (jinak se náklad počítá 2×).
- Dotazy na pohledávky (unpaid/overdue/aging/cashflow) musí mít guard `(invoice_type NOT IN ('invoice','proforma') OR amount_to_pay > 0)` — finální doklad ze zaplacené proformy má `amount_to_pay = 0`.

### Multiplatformnost (Windows / Linux / Docker)
- Veškerý kód musí být spustitelný a testovatelný na **Windows (IIS), Linuxu i v Dockeru** — žádná platformně specifická zkratka, která jinde rozbije běh.
- Proto jsou pomocné skripty záměrně **„zdvojené"**: ke každému `.sh` existuje ekvivalentní `.ps1`/`.cmd` (typicky v `cmd/`). Při změně jednoho udržuj v synchronizaci i druhý; nový skript přidávej rovnou v obou variantách.
- Webserver konfigurace existuje paralelně pro **Apache (`.htaccess`) i IIS (`web.config`)** — změny rewrite pravidel, hlaviček apod. promítej do obou.
- V PHP nepředpokládej konkrétní oddělovač cest ani casing souborového systému (viz guardy níže); rozdíly platforem řeš v kódu, ne podmínkou „jen na Windows".

### Runtime cesty a bezpečnost
- Cesty do `storage/` a `log/` vždy přes `RuntimePaths` (respektuje `MYINVOICE_DATA_DIR`), nikdy `Bootstrap::rootDir()`. Statické assety zůstávají na root dir.
- Path-traversal guardy musí být case-insensitive (Windows `realpath()` vrací nekonzistentní casing — porovnávej `strtolower` obě strany).
- Citlivé údaje (hesla, API klíče, connection stringy) nikdy do kódu, testů ani dokumentace.

### Frontend
- Po každé změně ve `web/src` spusť `pnpm build` — `dist/` je to, co se nasazuje a testuje; samotný `vue-tsc` nestačí.
- Drž se existujícího design language (sjednocené boxy, status badges, mobile cards) — před vymýšlením nového vzoru se podívej, jak to dělají sousední stránky.

## Testy

- PHPUnit 13, testy v `api/tests/{Unit,Integration,Architecture}`. Nové chování pokrývej testem; PR nesmí rozbít existující testy.
- **Pouze syntetická testovací data** — repo je veřejné. Žádné reálné doklady, výpisy, IBANy, čísla dokladů ani identifikátory skutečných protistran.
- České bankovní účty v testech musí projít mod-11 validací; ověřený placeholder: `1000000005 / 0100`.
- ISDOC export se validuje proti oficiálnímu XSD (`api/xsd/isdoc-invoice-6.0.2.xsd`).

## Manuál

- Zdroj v `manual/*.md` (česky). Při změně funkcionality viditelné uživatelem aktualizuj příslušnou kapitolu.
- Piš **jen aktuální stav** — žádné „od verze X.Y.Z", žádné odkazy na historii vývoje.
- Po změně Markdownu regeneruj výstupy: `php tools/generateManualHtml.php` + `php tools/exportManualToPdf.php`.
- Vzhled manuálu (`manual/manual.css`) zrcadlí design tokeny aplikace (`web/src/styles/main.css`) včetně dark mode — při změně tokenů udržuj synchronizaci.

## Konvence

- Drž se stylu okolního kódu (pojmenování, idiomy, hustota komentářů). Nepřidávej komentáře, které kód jen opakují.
- Commit messages česky, conventional-commits styl: `feat(scope): …`, `fix(scope): …`, `release: X.Y.Z — …` (viz `git log`).
- Změny v `CHANGELOG.md` a `VERSION` dělá maintainer při release — v běžném PR na ně nesahej.
- Necommituj vygenerované artefakty mimo zavedené výjimky (`dist/`, `manual/generated/` jsou commitované záměrně).

---
> Source: [radekhulan/myinvoice](https://github.com/radekhulan/myinvoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
