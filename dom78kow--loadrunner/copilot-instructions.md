## loadrunner

> Load Runner 2D — prosta gra platformowa/zręcznościowa w Pythonie + Pygame.

# Kontekst projektu

Load Runner 2D — prosta gra platformowa/zręcznościowa w Pythonie + Pygame.
Gracz przechodzi 10 map kafelkowych: omija przeciwników, korzysta z drabin,
zbiera klucz i otwiera drzwi do kolejnego poziomu.

## Stos technologiczny
- Python 3.13 (rozwijane na 3.13.7, wymagane 3.x)
- pygame 2.6.1 (SDL 2.28.4) — jedyna zależność zewnętrzna
- Brak C/C++, brak systemu budowania, brak frameworka testowego

## Struktura projektu
```text
LoadRunner.py       # CAŁA gra: zasoby, logika, pętla główna, renderowanie
maps/               # map1.txt–map10.txt, siatki tekstowe 15x9
images/player/      # sprite'y 32x32: idle_{dir}.png, walk{1,2,3}_{dir}.png
images/icons/       # kafelki 40x40 i ikony HUD 24x24
sounds/             # muzyka tła + door_open.wav
architecture.md     # opis mechanik i przepływu
```
Kierunki (`dir`): `up`, `down`, `left`, `right`.

## Format map
Siatka znaków, jeden znak = jeden kafel. Walidowana przy ładowaniu przez
`validate_map()`: mapa musi być prostokątna, mieć dokładnie jedno `S`,
co najmniej jedno `D` i nie mniej `K` niż `D`.

| Znak | Znaczenie |
| --- | --- |
| `0` | wolna przestrzeń / podłoga |
| `1` | ściana blokująca ruch |
| `S` | punkt startowy gracza (dokładnie jeden) |
| `K` | klucz do zebrania |
| `D` | drzwi do następnego poziomu (zużywają klucz) |
| `L` | drabina (ruch pionowy) |
| `E` | przeciwnik patrolujący poziomo |
| `G` | przepaść — wejście na nią zabija, trzeba ją przeskoczyć |

## Konwencje kodowania
- Python: `snake_case`, `UPPER_CASE` dla stałych ścieżek; bez type hints i formattera
- Stan rozgrywki żyje w dataclassach `Level` i `Player` (globalne `level`
  i `player`, tworzone dopiero w `start_level()`). Nowy stan dokładaj jako pole
  jednej z nich, nie jako kolejną globalną
- `start_level()` przenosi istniejącego `Player` na nowy spawn zamiast tworzyć
  go od nowa — inaczej życia i klucze zerują się przy każdym poziomie
- Komentarze i komunikaty po angielsku (starsze fragmenty są po polsku)
- Docstringi jednolinijkowe, tylko gdy logika nie jest oczywista
- Wyłącznie ASCII w `print()` — konsola Windows używa cp1250, emoji wywala
  `UnicodeEncodeError` przy przekierowanym stdout
- Ścieżki do zasobów zawsze przez `BASE_DIR` / `os.path.join`, nigdy względne
- Błąd krytyczny: `print("ERROR: ...")`, potem `pygame.quit()` i `sys.exit(1)`

## Budowanie i testowanie
```bash
# Uruchomienie
pip install pygame
python LoadRunner.py

# Smoke test bez okna i dźwięku (kod 124 = gra wystartowała i działa)
SDL_VIDEODRIVER=dummy SDL_AUDIODRIVER=dummy timeout 6 python LoadRunner.py
```
Brak zestawu testów. Moduł da się jednak zaimportować bez uruchamiania gry
(pętla siedzi w `main()` za guardem `__main__`), więc logikę testuje się tak:

```python
import LoadRunner as L
L.start_level(1)          # tworzy L.level i L.player
L.is_supported(L.player.rect)
```

Import nadal ładuje zasoby i otwiera okno `1×1` — pod `SDL_VIDEODRIVER=dummy`
to nie przeszkadza.

## Zasady
- Nie modyfikuj: zasobów w `images/` i `sounds/`
- Współrzędne rozgrywki liczone w skali 1x; `window_scale` skaluje tylko
  finalny obraz — nie mieszaj tych dwóch układów
- Mapa jest przesunięta w dół o `hud_height`; przeliczając piksele na kafle
  zawsze odejmij `hud_height` od współrzędnej Y
- Zawsze: uruchom smoke test po zmianach w logice
- Aktualizuj `architecture.md`, gdy zmieniasz mechanikę lub format map
- Pracuj na minimalnej mozliwej liczbie tokenow
- Rozbijaj zadania na male taski

---
> Source: [dom78kow/LoadRunner](https://github.com/dom78kow/LoadRunner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
