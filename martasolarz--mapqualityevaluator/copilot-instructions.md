## mapqualityevaluator

> To repozytorium to frament mojego projektu doktorskiego. Celem tej części doktoratu jest stworzenie modelu, który będzie oceniał jakość map statystycnzych

To repozytorium to frament mojego projektu doktorskiego. Celem tej części doktoratu jest stworzenie modelu, który będzie oceniał jakość map statystycnzych 
wg kryteriów kartograficznych. 

Cele tej części projektu doktorskiego:
	A) Formalizacja zasad metodyki kartografii tematycznej dla map statystycznych i opracowanie kryteriów oceny jakości tych map (P1, H1).
	B) Opracowanie metody zaprogramowania modelu do weryfikowania jakości map statystycznych z uwzględnieniem zasad kartografii tematycznej (P3, H3). 
	C) Opracowanie prototypu tego modelu, określenie jego jakości na podstawie zbioru testowego i udostępnienie modelu (P3, H3).

Jak ma działać model z perspektywy użytkownika:
1. Użytkownik wgrywa do modelu mapę statystyczną w formacie pdf/jpg/png
2. Model przetwarza mapę - powiedzmy do 5 minut
3. Użytkownik otrzymuje informację zwrotną:
	- Wskaźnik procentowy jakości mapy - gdzie 100% będzie oznaczać mapę w pełni zgodną z zasadami redakcji map w kartografii tematycznej
	- Informację o elementach wymagających poprawy (co obniżyło notę ogólną modelu dla tej mapy)


Kryteria dostępne są w pliku Podejscie_teoretyczne.md

W pierwszym kroku przeprowadziłam pozyskanie datasetu map statystycznych z MapPool - patrz katalog MapPool, dane zapisane są na /Volume/phd/data

---
> Source: [MartaSolarz/MapQualityEvaluator](https://github.com/MartaSolarz/MapQualityEvaluator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
