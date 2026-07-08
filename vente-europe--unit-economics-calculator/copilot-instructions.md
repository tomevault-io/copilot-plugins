## unit-economics-calculator

> Zbuduj single-page kalkulator unit economics dla Amazon US FBA seller.

# Amazon FBA Unit Economics Calculator — US Market

## Cel projektu

Zbuduj single-page kalkulator unit economics dla Amazon US FBA seller.
Użytkownik wpisuje wymiary i wagę produktu → kalkulator automatycznie klasyfikuje size tier
i oblicza FBA fee z wbudowanej tabeli.
Pozostałe koszty wpisywane ręcznie → kalkulator pokazuje profit, margin, ROI w czasie rzeczywistym.

Rynek: **Amazon US** (bez VAT, waluta USD).

---

## STATUS: Multi-market (US + EU) — 2026-07-03

Kalkulator obsługuje teraz **6 rynków** przez dropdown na górze: **US · DE · FR · IT · ES · UK**.
Jeden opublikowany `index.html`, przełącznik zmienia tabele fee, walutę, logikę size-tier i etykiety.

- **US** — bez zmian: logika Small/Large Standard (imperial wewnętrznie), price-tier, proximity gauges, weight-band optimiser.
- **EU (DE/FR/IT/ES/UK)** — osobny silnik metryczny:
  - Źródło danych: **`260410-FBA-Rate-Card-DE.pdf`** (Amazon "Tarifübersicht — Gebühren für Europa", ważne od 17 kwietnia 2026). PDF zawiera WSZYSTKIE kraje w jednym dokumencie; liczby identyczne w wersji DE/EN/PL. Jest też kopia PL w folderze.
  - **DE = kolumna CEP (DE/PL/CZ)** (wybór Toma), NIE "Nur DE".
  - Programy: **Standard (str. 6)** + **Low-Price (str. 5)** — toggle widoczny tylko dla EU. Low-Price = przedmioty ≤ £20/€20 (z VAT), max Small parcel ≤ 400 g.
  - Tiery metryczne: koperty (Umschlag) → paczki (Paket) → oversize. Waga wolumetryczna = **L×W×H ÷ 5000**. Dla paczek billable = max(actual, volumetric); dla kopert tylko actual.
  - Oversize → ręczne wpisanie fee (jak US).
  - **Referral fee liczona od GROSS (z VAT)** — Amazon EU nalicza Empfehlungsgebühr od Angebotspreis (cena brutto), NIE od netto (str. 24 PDF). Dlatego dla EU jest pole **Amazon Price (gross)** + **VAT %** (default per kraj: DE 19, FR 20, IT 22, ES 21, UK 20 — edytowalne, bo reduced rates dla food/books/health). User wpisuje gross → **Selling Price (net)** auto = gross/(1+VAT); pola zlinkowane dwukierunkowo (`onGrossInput/onNetPriceInput/onVatInput`, `EU_VAT`). Referral = gross × ref%. Reszta profit-math na netto. Break-even EU liczony osobno (referral skaluje z gross, PPC z net). PPC/ACoS zostaje na netto (Tom prosił tylko o referral). US bez zmian (referral od net, brak VAT).
  - Referral % — nadal ręczne pole (default 15%), wspólne dla obu rynków.
  - Uwaga: Amazon dolicza **+1.5% fuel & logistics surcharge** na shipping fees od 17 Apr 2026 — traktujemy liczby z tabeli jako bazę, weryfikować z fakturą.
- **Silnik EU zweryfikowany** standalone (9/9 testów) + integracja przez DOM-harness (US regression + DE/UK/Low-Price). Dane EU zaszyte w `index.html` jako `EU_STANDARD_ENVELOPE/PARCELS`, `EU_LOWPRICE_*`. Funkcje: `euFee()`, `computeEuFba()`, `onMarketChange()`, `setProgram()`.
- **Bugfix przy okazji:** oversize NIE ustawia już `fbaFeeManualOverride=true` (US i EU) — auto-fill wraca gdy produkt przestaje być oversize. Override ustawia tylko ręczna edycja pola FBA Fee.

- **EU Size Tier Proximity** (2026-07-03) — zbudowany osobny widok dla EU (`euProximitySection`, funkcje `updateEuProximity()`, `euBandsForTier()`, `euRenderBands()`, `EU_ZONES`). Pokazuje: 4 gauge (L/W/H/waga) z headroom do limitu tieru (container zones: Envelope 33×23×6/0.96kg → Small parcel 35×25×12/3.9kg → Standard parcel 45×34×26/11.9kg), chip driver (📐 volumetric / ⚖️ actual), linię "next tier up", oraz strip fee-by-weight-band (drop-to-cheaper / current / next-band cost + headroom). US gauges (`proximitySection`) chowane dla EU i odwrotnie. `euFee()` zwraca teraz też `dims` + `isParcel`.

- **EU Monthly Storage (2026-07-08)** — auto-liczony magazyn/szt./mies. = objętość pudełka × stawka. Dane `EU_STORAGE` (rate card str. 17): EUR (DE/FR/IT/ES) €/m³, UK £/ft³, każda `[Sty–Wrz, Paź–Gru]`, klasy: standard / apparel / oversize. Objętość = L×W×H/1e6 m³ (UK ×35.3147 na ft³). Klasa oversize brana z `res.tier==='oversize'`. UI: `euStorageControls` (select sezon + kategoria) w sekcji Koszty, `storageHint`, auto-fill pola `storage` (jak FBA fee; `storageManualOverride`, `onStorageManual/onStorageParamChange`, `euStorageFee()`). Dangerous goods pominięte. Weryfikacja: DE 25.4×17.8×7.6 → €0.09 (Sty–Wrz) / €0.18 (Paź–Gru), UK ft³ OK, override trzyma.

### Do zrobienia w przyszłości (jeśli Tom poprosi)
- EU oversize tiers (base + per-kg) — obecnie manual.
- Storage: dangerous goods + Lagernutzungszuschlag (utilization surcharge) — obecnie pominięte.
- Kolejne rynki (NL/SE/PL/BE) — dane są w tym samym PDF, wystarczy dodać kolumny.
- Referral % auto z kategorii (tabele str. 24-26).

---

## Stack

- **Jeden plik `index.html`** — HTML + CSS + JS inline, zero zależności zewnętrznych
- Zero npm, zero frameworków, zero build step
- Działa po otwarciu w przeglądarce offline
- Gotowy do deploy na GitHub Pages bez żadnej konfiguracji

---

## Struktura pól kalkulatora

### Sekcja 1: REVENUE
| Pole | Typ | Placeholder | Opis |
|------|-----|-------------|------|
| Selling Price | number, $ | 29.99 | Cena na listingu Amazon US |
| Units Sold / Month | number | 100 | Do wyliczeń miesięcznych (opcjonalne) |

### Sekcja 2: FBA FEE CALCULATOR
Użytkownik wpisuje wymiary → kalkulator automatycznie wylicza fee.

| Pole | Typ | Placeholder | Opis |
|------|-----|-------------|------|
| Length (longest side) | number, inches | 10 | Najdłuższy bok opakowania |
| Width (median side) | number, inches | 7 | Środkowy bok |
| Height (shortest side) | number, inches | 2 | Najkrótszy bok |
| Weight | number, lbs | 0.8 | Waga jednostki z opakowaniem |
| Price tier | select | $10–$50 | Wpływa na FBA fee — 3 opcje: <$10 / $10–$50 / >$50 |
| Rate year | toggle | 2025 | 2025 rates / 2026 rates — przełącza kolumny tabeli |

Po wpisaniu danych wyświetl pod polami (read-only):
- **Detected tier:** Small Standard / Large Standard / Oversize (ręczne)
- **Shipping weight:** X oz lub X lb
- **FBA Fee:** $X.XX (auto)

Pole **FBA Fee** w sekcji kosztów powinno być auto-wypełniane z powyższego,
ale pozostać edytowalne ręcznie (na wypadek oversize lub korekty).

### Sekcja 3: COSTS
| Pole | Typ | Placeholder | Opis |
|------|-----|-------------|------|
| COGS / unit | number, $ | 5.00 | Koszt produktu od dostawcy |
| Inbound Shipping / unit | number, $ | 1.20 | Transport do Amazon FC per unit |
| Referral Fee | number, % | 15 | Domyślnie 15%, edytowalne |
| FBA Fee | number, $ | auto | Auto z sekcji 2, edytowalne |
| Storage Fee / unit / month | number, $ | 0.20 | Opcjonalne |
| PPC Spend / unit | number, $ | 1.50 | Advertising cost per unit sold |
| Other Costs / unit | number, $ | 0.30 | Inserty, prep, fotografia, etc. |

### Sekcja 4: RESULTS (tylko output, bez inputów)
- Net Profit / unit ($)
- Profit Margin (%)
- ROI (%)
- Break-even Price ($)
- Monthly Profit ($) — tylko jeśli podano units
- Waterfall kosztów
- Visual cost breakdown bar

---

## Logika FBA Fee — implementacja krok po kroku

### Krok 1: Klasyfikacja size tier

Sprawdzaj warunki **w tej kolejności**:

```javascript
function classifyTier(length, width, height, weight_lbs) {
  // Upewnij się że L >= W >= H (posortuj malejąco)
  const dims = [length, width, height].sort((a, b) => b - a);
  const L = dims[0], W = dims[1], H = dims[2];
  const weight_oz = weight_lbs * 16;

  // Small Standard: wszystkie warunki muszą być spełnione
  if (weight_oz <= 16 && L <= 15 && W <= 12 && H <= 0.75) {
    return 'small-standard';
  }

  // Large Standard: wszystkie warunki muszą być spełnione
  if (weight_lbs <= 20 && L <= 18 && W <= 14 && H <= 8) {
    return 'large-standard';
  }

  // Wszystko powyżej = Oversize (kalkulator nie obsługuje)
  return 'oversize';
}
```

### Krok 2: Shipping weight

```javascript
function calcShippingWeight(length, width, height, weight_lbs, tier) {
  if (tier === 'small-standard') {
    // Small Standard: używa actual weight (w oz)
    return { value: weight_lbs * 16, unit: 'oz' };
  }

  if (tier === 'large-standard') {
    // Large Standard: max(actual weight, dimensional weight)
    const dimWeight_lbs = (length * width * height) / 139;
    const shipping_lbs = Math.max(weight_lbs, dimWeight_lbs);
    return { value: shipping_lbs, unit: 'lbs' };
  }
}
```

### Krok 3: Lookup tabeli fee

Zaimplementuj tę tabelę jako stałą w JS (dane dokładnie z tabeli poniżej).

---

## Tabela FBA Fee — dane do wbudowania w JS

**Źródło: Amazon US, oficjalna tabela fee.**

### Small Standard — shipping weight w OZ

```javascript
const FBA_SMALL_STANDARD = [
  // { maxOz, fees: { '<10': {2025, 2026}, '10-50': {2025, 2026}, '>50': {2025, 2026} } }
  { maxOz: 2,  fees: { '<10': [2.29, 2.43], '10-50': [3.06, 3.32], '>50': [3.06, 3.58] } },
  { maxOz: 4,  fees: { '<10': [2.38, 2.49], '10-50': [3.15, 3.42], '>50': [3.15, 3.68] } },
  { maxOz: 6,  fees: { '<10': [2.47, 2.56], '10-50': [3.24, 3.45], '>50': [3.24, 3.71] } },
  { maxOz: 8,  fees: { '<10': [2.56, 2.66], '10-50': [3.33, 3.54], '>50': [3.33, 3.80] } },
  { maxOz: 10, fees: { '<10': [2.66, 2.77], '10-50': [3.43, 3.68], '>50': [3.43, 3.94] } },
  { maxOz: 12, fees: { '<10': [2.76, 2.82], '10-50': [3.53, 3.78], '>50': [3.53, 4.04] } },
  { maxOz: 14, fees: { '<10': [2.83, 2.92], '10-50': [3.60, 3.91], '>50': [3.60, 4.17] } },
  { maxOz: 16, fees: { '<10': [2.88, 2.95], '10-50': [3.65, 3.96], '>50': [3.65, 4.22] } },
];
// fees[priceTier][0] = 2025, fees[priceTier][1] = 2026
```

### Large Standard — shipping weight w OZ (do 16oz) i LBS (powyżej)

```javascript
const FBA_LARGE_STANDARD = [
  // Wagi w oz (do 1 lb)
  { maxOz: 4,  fees: { '<10': [2.91, 2.91], '10-50': [3.68, 3.73], '>50': [3.68, 3.99] } },
  { maxOz: 8,  fees: { '<10': [3.13, 3.13], '10-50': [3.90, 3.95], '>50': [3.90, 4.21] } },
  { maxOz: 12, fees: { '<10': [3.38, 3.38], '10-50': [4.15, 4.20], '>50': [4.15, 4.46] } },
  { maxOz: 16, fees: { '<10': [3.78, 3.78], '10-50': [4.55, 4.60], '>50': [4.55, 4.86] } },

  // Wagi w lbs (powyżej 1 lb) — maxLbs
  { maxLbs: 1.25, fees: { '<10': [4.22, 4.22], '10-50': [4.99, 5.04], '>50': [4.99, 5.30] } },
  { maxLbs: 1.5,  fees: { '<10': [4.60, 4.60], '10-50': [5.37, 5.42], '>50': [5.37, 5.68] } },
  { maxLbs: 1.75, fees: { '<10': [4.75, 4.75], '10-50': [5.52, 5.57], '>50': [5.52, 5.83] } },
  { maxLbs: 2,    fees: { '<10': [5.00, 5.00], '10-50': [5.77, 5.82], '>50': [5.77, 6.08] } },
  { maxLbs: 2.25, fees: { '<10': [5.10, 5.10], '10-50': [5.87, 5.92], '>50': [5.87, 6.18] } },
  { maxLbs: 2.5,  fees: { '<10': [5.28, 5.28], '10-50': [6.05, 6.10], '>50': [6.05, 6.36] } },
  { maxLbs: 2.75, fees: { '<10': [5.44, 5.44], '10-50': [6.21, 6.26], '>50': [6.21, 6.52] } },
  { maxLbs: 3,    fees: { '<10': [5.85, 5.85], '10-50': [6.62, 6.67], '>50': [6.62, 6.93] } },

  // 3+ lb to 20 lb: base + $0.08 per każde rozpoczęte 4 oz powyżej 3 lb
  // base fees dla 3+ lb:
  // 2025: <$10 = $6.15 | $10-$50 = $6.92 | >$50 = $6.92
  // 2026: <$10 = $6.15 | $10-$50 = $6.97 | >$50 = $7.23
];

const FBA_LARGE_OVER3LB_BASE = {
  '<10':  [6.15, 6.15],
  '10-50': [6.92, 6.97],
  '>50':  [6.92, 7.23],
  // [0] = 2025, [1] = 2026
};

function calcOver3lb(shipping_lbs, priceTier, yearIndex) {
  const base = FBA_LARGE_OVER3LB_BASE[priceTier][yearIndex];
  const ozAbove3lb = (shipping_lbs - 3) * 16;
  const intervals = Math.ceil(ozAbove3lb / 4); // każde rozpoczęte 4 oz
  return base + (intervals * 0.08);
}
```

### Lookup function

```javascript
function getFBAFee(tier, shipping_weight, priceTier, year) {
  // year: '2025' lub '2026'
  const yearIndex = year === '2025' ? 0 : 1;

  if (tier === 'oversize') return null; // ręczne wpisanie

  if (tier === 'small-standard') {
    // shipping_weight w oz
    const row = FBA_SMALL_STANDARD.find(r => shipping_weight <= r.maxOz);
    if (!row) return null;
    return row.fees[priceTier][yearIndex];
  }

  if (tier === 'large-standard') {
    const shipping_oz = shipping_weight * 16; // shipping_weight jest w lbs

    if (shipping_weight > 3) {
      return calcOver3lb(shipping_weight, priceTier, yearIndex);
    }

    // Szukaj w tabeli — najpierw oz (do 1 lb), potem lbs
    for (const row of FBA_LARGE_STANDARD) {
      if (row.maxOz && shipping_oz <= row.maxOz) {
        return row.fees[priceTier][yearIndex];
      }
      if (row.maxLbs && shipping_weight <= row.maxLbs) {
        return row.fees[priceTier][yearIndex];
      }
    }
  }

  return null;
}
```

---

## Formuły kalkulatora

```javascript
referral_fee     = selling_price * (referral_pct / 100)   // od pełnej ceny!
total_costs      = cogs + inbound + referral_fee + fba_fee + storage + ppc + other
profit_per_unit  = selling_price - total_costs
margin_pct       = (profit_per_unit / selling_price) * 100
roi_pct          = (profit_per_unit / total_costs) * 100
breakeven_price  = total_costs / (1 - referral_pct / 100)
monthly_profit   = profit_per_unit * units_per_month  // jeśli podano units
```

---

## UX wymagania

1. **Live update** — wyniki odświeżają się przy każdym keystroke, bez przycisku "Oblicz"
2. **FBA auto-fill** — po wpisaniu wymiarów i wagi: pole FBA Fee wypełnia się automatycznie + wyświetla detected tier i shipping weight
3. **Oversize warning** — czerwony komunikat jeśli wymiary > Large Standard + odblokowanie ręcznego pola FBA fee
4. **Rate year toggle** — przycisk "Jan 2025 – Oct 2025" / "From Jan 2026" przełącza kolumny tabeli
5. **Price tier auto-detect** — jeśli selling price jest wpisane, price tier ustawiaj automatycznie (`<$10`, `$10–$50`, `>$50`) z możliwością ręcznego override
6. **Waterfall** — lista: Selling Price → − Referral Fee → − FBA Fee → − COGS → − Inbound Shipping → − Storage → − PPC → − Other → = Net Profit
7. **Cost breakdown bar** — wizualny podział procentowy każdego kosztu względem selling price
8. **3 preset scenariusze** — przyciski z przykładowymi produktami do szybkiego testowania kalkulatora
9. **Stan pusty** — czytelny placeholder (nie zera) gdy pola są puste
10. **Estetyka** — ciemny motyw, profesjonalny, minimalistyczny

---

## Przykłady testowe — sprawdź po zbudowaniu

| Produkt | L×W×H (in) | Weight | Price | Price tier | Year | Expected tier | Expected shipping weight | Expected FBA fee |
|---------|-----------|--------|-------|-----------|------|--------------|--------------------------|-----------------|
| Mały suplement | 5×3×0.5 | 0.4 lb (6.4 oz) | $24.99 | $10–$50 | 2025 | Small Standard | 6.4 oz → row 6-8 oz | $3.33 |
| Sredni produkt | 10×7×3 | 1.2 lb | $34.99 | $10–$50 | 2025 | Large Standard | dim=1.52 lb → max(1.2, 1.52)=1.52 lb → 1.5-1.75 lb | $5.52 |
| Ciezszy | 12×8×4 | 2.5 lb | $49.99 | $10–$50 | 2025 | Large Standard | dim=2.77 lb → max(2.5, 2.77)=2.77 lb → 2.75-3 lb | $6.62 |
| 4 lb produkt | 14×10×5 | 4.0 lb | $39.99 | $10–$50 | 2025 | Large Standard | 4.0 lb > 3 lb → calcOver3lb | $6.92 + ceil(16/4)×$0.08 = $6.92+$0.32 = $7.24 |
| Tani produkt | 5×3×0.5 | 0.3 lb (4.8 oz) | $8.99 | <$10 | 2026 | Small Standard | 4.8 oz → row 4-6 oz | $2.56 |

---

## Pliki do stworzenia

1. `index.html` — kompletny kalkulator
2. `README.md` — opis projektu + instrukcja GitHub Pages deploy

---

## Kolejność implementacji

1. Napisz i przetestuj funkcje JS: `classifyTier()`, `calcShippingWeight()`, `getFBAFee()` — zweryfikuj przykładami testowymi w konsoli przeglądarki
2. Zbuduj HTML strukturę z wszystkimi sekcjami i polami
3. Podłącz logikę FBA do UI (auto-fill + detected tier display)
4. Zaimplementuj formuły kalkulatora i live update
5. Dodaj waterfall i cost breakdown bar
6. Dodaj rate year toggle, price tier auto-detect, preset scenariusze
7. Dopracuj estetykę i responsywność
8. Utwórz README.md

---
> Source: [vente-europe/unit-economics-calculator](https://github.com/vente-europe/unit-economics-calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
