## ledger-app

> Investiční portfolio tracker nad `unified_format_raw`.

# CLAUDE.md – Ledger App

## Projekt
Investiční portfolio tracker nad `unified_format_raw`.
Desktop aplikace v Pythonu s Flet UI.

## Jednovětá esence
Aplikace je nástroj pro čtení a vytváření tokového ledgeru, nikoli pro jeho interpretaci; pravda je v datech, nikoli v kódu.

## Architektura – 4 vrstvy
```
I/O Modul  →  CORE (doménová logika)  →  Workflow (UI logika)  →  Grafika (Flet)
```
Komunikace vždy jedním směrem. Vrstvy se nepřeskakují.

## Datový model: unified_format_raw
Každý řádek = jeden atomický tok aktiva.

| Pole | Typ | Popis |
|------|-----|-------|
| id | String | Interní identifikátor (sdílený double-entry párem) |
| timestamp | ISO 8601 | Čas toku |
| type | Enum | BUY, SELL, TRANSFER, FEE, REVERSAL |
| asset | String | Aktivum |
| amount | Decimal | Množství (znaménko = směr: + příchozí, – odchozí) |
| currency | String | Protistranná měna |
| price | Decimal? | Informativní jednotková cena |
| venue | String | Místo toku (burza, wallet – vše je jen venue) |
| note | String? | Volitelná poznámka |

RAW neobsahuje žádná externí ID. Reference patří do note.

## Kanonické principy (neměnné)
1. **Ledger-centric** – jediný zdroj pravdy je tokový ledger
2. **RAW je datová vrstva** – načíst, validovat, nikdy zpětně měnit
3. **Dva režimy** – Import (read-only) a Operace (append přes UI)
4. **Append-only** – žádné mazání/přepisování, oprava = REVERSAL
5. **Modulární** – každá část má jednu odpovědnost
6. **Funguje bez dat** – prázdný ledger je validní stav
7. **Venue-agnostic** – burza = wallet = banka = jen uzel toku
8. **Budoucnost bez refaktoringu** – tokový model se nemění

## Validace vs. diagnostika
- **Validator** = syntaktická kontrola (povinná pole, typy, povolené hodnoty). Nekontroluje sémantiku.
- **Diagnostika** = výpočtová vrstva. Může spočítat stav, označit záporný zůstatek, zobrazit warning. Nikdy neblokuje zápis.

## Deduplikace
```
row_fp = sha256(timestamp + type + venue + asset + amount)
```
Unikátní index v DB. Opakovaný import = 0 duplicit.

## REVERSAL
Oprava = nový tok s opačným znaménkem amount, type=REVERSAL, note odkazuje na původní záznam.

## Struktura projektu
```
ledger_app/
├── main.py                 # Entry point
├── core/
│   ├── model.py            # RawRow dataclass
│   ├── validator.py        # Syntaktická validace
│   └── ledger_store.py     # SQLite append-only DB
├── io_module/
│   └── raw_loader.py       # Načítání *_raw.xlsm / .csv
├── ui/
│   └── terminal.py         # Terminálové UI (→ nahradit Flet)
└── test_mvp.py             # Testy MVP kritérií
```

## Technologie
- Python 3.10+
- SQLite (Ledger Store)
- openpyxl (čtení .xlsm)
- Flet (UI – zatím terminálové, přechod na Flet je hlavní další krok)

## Co MVP řeší ✓
- Import hotových *_raw souborů (xlsm, csv)
- Deduplikace při importu (sha256 fingerprint)
- Append-only ledger v SQLite
- Timeline, asset view, venue view
- Ruční zápis toku přes UI
- REVERSAL opravné toky
- Diagnostika záporných zůstatků (warning, ne blokace)
- Prázdný ledger = validní stav

## Co MVP vědomě neřeší ✗
- Žádné importy z burz (parsování cizích formátů)
- Žádné ukládání stavů (vše odvozené)
- Žádné automatické opravy
- Žádná daňová legislativa

## Další kroky
1. **Přechod na Flet UI** – nahradit ui/terminal.py za Flet grafiku
2. **Parsery pro burzy** – I/O modul: Anycoin, Bybit, Kraken, Revolut
3. **Export** – CSV / RAW výstupy (stavy portfolia, cashflow, P&L)

## Konvence
- Venue vždy lowercase
- Asset vždy uppercase
- Timestamp vždy ISO 8601
- Amount s pevnou decimal přesností
- Double-entry: BUY/SELL = 2 řádky se stejným id
- Žádné UPDATE/DELETE v DB – append-only

---
> Source: [herczegh-star/LEDGER_APP](https://github.com/herczegh-star/LEDGER_APP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
