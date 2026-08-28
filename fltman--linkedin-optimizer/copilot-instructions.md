## linkedin-optimizer

> Läs [CONTRACT.md](CONTRACT.md) först. Den är bibeln: grundregler, API-form och de fällor

# CLAUDE.md — LinkedIn Optimizer

Läs [CONTRACT.md](CONTRACT.md) först. Den är bibeln: grundregler, API-form och de fällor
som faktiskt bet under bygget. Kod som bryter mot kontraktet är fel även om den fungerar.

## Snabbstart

```bash
tools/serve.sh          # http://localhost:8175
php tools/test.php      # 182 tester — servern
node tools/test.mjs     # 80 tester — de rena funktionerna i app.js
```

Ingendera rör nätet eller `site/data`.

## Det viktigaste att inte glömma

- **Nyckeln passerar aldrig servern.** Klienten pratar direkt med OpenRouter. Lägg aldrig
  tillbaka en PHP-proxy — då ser servern både nyckeln och fotot igen. `test.mjs` vaktar det.
- **Reasoning måste vara AV på regissörsanropet.** En resonerande modell äter hela
  `max_tokens` på att tänka och svarar med `content: null`. Kostade 31 öre för noll text.
- **Sajten spelar rakt.** Det genererade inlägget innehåller aldrig ironi och erkänner
  aldrig att det är ett skämt. Chrome runtomkring får vara torrt lustig — aldrig inlägget.
- **Ett regissörsanrop, sedan ett bildanrop. Sekventiellt, och kedjat av klienten.**
  Regissören ser fotot, väljer röst och scen själv och skriver både inlägget och bildprompten.
  Bygg inte om till parallellt — då tappar bilden kopplingen till det texten fastnade vid.
  Inlägget ritas så fort det är skrivet, med en platshållare där bilden ska in.
- **Användaren väljer ingenting utom språk.** Inga stil- eller röstväljare i gränssnittet.
- **"Clean the frame, not the object."** Bilden ska vara påkostad — proffsigt ljus, krämig
  bokeh, färgsatt grade — men motivet måste gå att känna igen som just den användarens sak.
  Skada är identitet, inte defekt. Tre uppmätta fällor står i CONTRACT.md: motivet måste
  fotograferas OM i scenen (annars blir det ett platt urklipp), å/ä/ö får aldrig räknas upp
  (modellen ritar bokstäverna), och ljus beskrivs genom sin verkan (annars syns softboxen).
- **Biblioteket är inspiration, inte regler.** Rubrikerna i regissörsprompten säger det
  uttryckligen. Utan den märkningen väljer modellen alltid exakt en av de fyra, och då är vi
  tillbaka i lådorna vi tog bort.
- **Ingen databas, inga npm-paket, inget byggsteg, inga CDN.** Målet är att hela sajten
  går att FTP:a till ett delat webbhotell.
- **Svenska och engelska är två röster, inte en översättning.** Och alltid korrekta å ä ö,
  även i kod och kommentarer. Instruera subagenter uttryckligen om det.
- **Lökighetsmätaren är kalibrerad mot två ankare** (lökigt ≥ 70, platt < 30, minst 45
  isär). Rör du viktningen måste båda hålla, annars mäter mätaren bara sig själv.
- **Bildprompternas injektionsskydd står kvar.** Användartexten hamnar inuti bilden och
  inuti systemprompten; raden om att den bara är innehåll är det enda försvaret.

## Att lägga till en scen eller en röst

Allt bor i `inc/promptdata.php`. Lägg till nyckeln under `imageStyles` respektive `personas`.
En scen behöver `sceneSv` + `sceneEn` (korta beskrivningar som monteras in i regissörsprompten)
och `promptTemplate` (den långa engelska mallen som används som reservbildprompt). En röst
behöver `voiceSv` + `voiceEn`. `render_library()` plockar upp dem automatiskt — ingen annan fil
behöver ändras. Testsviten kontrollerar redan att hela biblioteket monteras in.

## Väggen

Delning är opt-in och tar med inlägget, den optimerade bilden, den ursprungliga meningen **och
originalfotot** — det sista bara för att kryssrutans etikett säger det rakt ut. Ändrar du vad
som publiceras måste etiketten ändras i samma commit, på båda språken.

`data/` är gitignorerad och innehåller riktig användardata. Radera aldrig i den utan att
fråga. Testsviten pekar om `DATA_DIR` till en temporär katalog just för att slippa det.

---
> Source: [fltman/linkedin-optimizer](https://github.com/fltman/linkedin-optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
