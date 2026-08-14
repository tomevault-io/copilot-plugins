## chaos

> 1 ÁLTALÁNOS SZABÁLYOK

1 ÁLTALÁNOS SZABÁLYOK

A) Csak sebészi módosítás engedélyezett:
- TÖRLÉS vagy PONTOS CSERE megengedett.
- Nincs újrafogalmazás, nincs tartalmi bővítés.
- A meglévő sorokat, bekezdéseket, struktúrát meg kell őrizni.
- Kövesd az .editorconfig kódolási/sorvég beállításait (UTF-8, CRLF).

B) Stíluspreferencia: Pragmatikus KISS + „move-fast” prototipizálás; minimalista util / UNIX-szerű „kis eszköz” szemlélet. A tömörség és egyszerűség elsődleges; a guardok és „keményítés” később, célzottan kerüljenek be.

C) Műveleti jelzők:
- ACK = módosítás engedélyezése;
- NOP = nincs módosítás, csak egyeztetés.

D) Parancsprotokoll (ACK/NOP + célzás):
- `ACK` = hajtsd végre a parancsot kérdés nélkül az irányelvek szerint, viszont csak külön kérésre kommitolj.
- `ACK:KOM` = alkalmazd a kommentelési szabályokat; fésüld át és írd át.
- `ACK:KOM:COL` = alparancs, pl. kommentek oszlopba rendezése blokkon belül (azonos oszlopra igazítva).
- Opcionális szűkítés: `ACK:KOM source.jl` csak a megadott fájlra.
- További szűkítés: `ACK:KOM add_source!` csak a megadott azonosítóra.
- Tartomány: `ACK:KOM source.jl 31:41` csak a megadott sorokra.
- `NOP:...` = csak javasolj, ne módosíts.

E) Kommitolási szabályok:
- Csak külön kérésre kommitolj.
- Csak a kért fájlokat stage-eld.
- Egy változás = egy kommit, ha logikailag független.
- Kommit üzenet egyeztetése kötelező.
- Kommit üzenet: technikai pontok, a végén 1 mondat felhasználói változás.

2 KÓDOLÁSI SZABÁLYOK, PARANCSOK

A) KOM - Kommentelési szabályok:
- magyarul kommentelj, ne tedd a parancskódot (KOM) a kommentbe.
- legyen tömör (≤100 karakter);
- egy soros komment megengedett blokk-szintű szerkezetek (függvények, ciklusok stb.) felett, ha az a változtatás kontextusát segíti;
- blokkon belül a kommenteket egy oszlopba rendezd (azonos oszlopba igazítva). Ha egy programsor túl hosszú, akkor az az egy komment eltérhet az oszloptól, emiatt ne módosuljon a 
- továbbá egy programutasítás sor végén elhelyezett rövid komment is megengedett.
- adott .jl file 1-3 sorában egy 3 soros rövid leírás, mire is jó adott .jl file.
- ne az épp aktuális átalakítást kommenteld, igyekezz inkább végleges kommentet adni.
teljes oszlop pozíciója.
- korábbi kommenteket is módosíthatod, átfogalmazhatod:
    - ahol ezt találod: # <rethink>, tedd meg, de akár változatlanul is hagyhatod;
    - ahol ezt találod: # <del>, az a kommentet töröld a # jellel együtt, viszont a tartalmát felhasználhatod, ha látsz benne fontos információt;
    - ahol ezt találod: # <kom>, oda írj egy találó kommentet.

B) Rövidzáras feltételek: egyszerű, egyutasításos esetekben használd az isnothing(x) || do_sg() / cond && action() mintát. Ha több utasítást kell feltételesen végrehajtani, használj if … end szerkezetet. Ne keverd === nothing-nel; az isnothing(x) a preferált.

C) INL - Egyszerhasználatos lokálisok kerülése: ahol olvasható, preferáld az inline konstrukciót (pl. add_source!(Source(...))).

D) Fail-fast elv: ne clampelj hívási pontokon; invariánsok ellenőrzése helper(ek)ben, debug módban @dbg_assert-tel (@static if DEBUG_MODE; @assert …; end). Release-ben 0 overhead.

E) Callback sorrend: az inicializáló állítások (pl. i_selected) után történjen a feliratkozás (on(...)).

F) Observable‑kötések: általános helper(ek)be szervezve, opcionális paraméterekkel (pl. mk_slider! → bind_to, transform).

G) GRD - Felesleges guardok kerülése: ha az értéktartományt a flow garantálja, ne tegyél @isdefined-et és ne clampelj; lásd 3 C).

H) LAN - Láncolt értékadás: használd rövid, mellékhatás‑mentes inicializálásoknál; pl. fig[1,1] = gl = GridLayout(). A kifejezés jobbra asszociatív, ezért ekvivalens a gl = GridLayout(); fig[1,1] = gl formával. Mellékhatásos vagy több lépcsős hívások láncolását kerüld, mert nehezíti a debuggolást.

3 RELEASE SZABÁLYOK

- Verzioszabaly: kezdes 0.1.0-alpha.1, majd 0.1.0-alpha.2, stabilnal 0.1.0; verziót emelni kell Manifest.toml-ban és Project.toml-ban
- Tag formatum: vX.Y.Z elotag, pl. v0.1.0-alpha.1
- Release tipus: alpha eseten Pre-release jeloles
- Minimum release note: 3 rovid blokk: "Uj", "Valtozas", "Ismert hibak"
- Assetek: csak futtathato/zip, forras automatikus GitHub-on
- Minimal ellenorzes: futtathato elindul-e + alap UI megnyithato
- Release elott: Infiltrator csomagot el kell tavolitani
- Release elott: a using Infiltrator sort ki kell kommentezni
- Release elott: a debug modot ki kell kapcsolni

---
> Source: [kriszo74/Chaos](https://github.com/kriszo74/Chaos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
