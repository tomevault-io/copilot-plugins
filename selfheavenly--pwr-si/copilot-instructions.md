## pwr-si

> **Schemat blokowy sterownika rozmytego i omówienie funkcji podstawowych elementów**

# Opracowane pytania teoretyczne z poprzednich kolokwiów

## Podaj schemat blokowy sterownika rozmytego. Omów funkcje podstawowych elementów sterownika.

**Schemat blokowy sterownika rozmytego i omówienie funkcji podstawowych elementów**

Sterowniki rozmyte (FLC – fuzzy logic controller) stanowią alternatywę dla sterowników klasycznych w systemach sterowania, szczególnie tam, gdzie kluczowa jest prostota rozwiązania i szybkość działania, a nie bezwzględna precyzja. Celem sterowania rozmytego jest przetwarzanie danych wejściowych na dane wyjściowe w celu osiągnięcia i utrzymania pożądanego stanu urządzenia lub systemu.

Schemat blokowy sterownika rozmytego składa się z czterech głównych modułów:

- **Moduł rozmywania** (Moduł rozmywania)
- **Moduł wnioskowania** (Moduł wnioskowania)
- **Moduł wyostrzania** (Moduł wyostrzania)
- **Baza reguł** (Baza reguł)

Funkcje podstawowych elementów sterownika rozmytego:

- **Moduł rozmywania:** Służy do **zastąpienia każdej "ostrej" (liczbowej) wartości wejściowej zbiorem rozmytym**. Jest to niezbędne, ponieważ moduł wnioskujący operuje na zbiorach rozmytych. Konkretne wartości liczbowe sygnałów wejściowych są zastępowane odpowiednimi wartościami zmiennych lingwistycznych, np. "mały", "średni", "duży". Często stosuje się zastąpienie każdej wartości ostrej odpowiednim singletonem rozmytym.
- **Baza reguł:** Jest to tzw. **model lingwistyczny procesu sterowania**. Zawiera zbiór reguł rozmytych, które opisują zależności między zmiennymi lingwistycznymi.
- **Moduł wnioskowania:** Jest **głównym elementem sterownika rozmytego**. Działa w oparciu o wybrany schemat wnioskowania przybliżonego, np. modus ponens. Sposób wyliczenia zbioru wynikowego (konkluzji) zależy od wyboru T-normy, przy czym w sterowaniu rozmytym najczęściej stosuje się operacje minimum lub iloczynu algebraicznego. Moduł ten dla każdej reguły z bazy określa stopień prawdziwości części przesłankowej (na podstawie stopnia prawdziwości każdego ze stwierdzeń), a następnie na tej podstawie określa stopień prawdziwości konkluzji.

Źródła nie opisują szczegółowo funkcji Modułu wyostrzania, choć jest on wymieniony w schemacie.

## Podaj definicję zbioru rozmytego. Omów 4 wybrane klasy funkcji przynależności dla zbiorów rozmytych.

**Definicja zbioru rozmytego i funkcje przynależności**

W życiu codziennym często występują pojęcia i zjawiska wieloznaczne i nieprecyzyjne, których nie można opisać za pomocą klasycznej teorii zbiorów i logiki dwuwartościowej. W przypadku przetwarzania wiedzy niepewnej lub niepełnej stosuje się teorię zbiorów rozmytych. Rozwój tej teorii zapoczątkował w 1965 roku Lofti Zadeh.

**Zbiorem rozmytym** A w pewnej niepustej przestrzeni X (A ⊆ X) nazywamy **zbiór par**:
A = {(x, µA(x); x ∈ X}
gdzie **µA : X → jest funkcją przynależności zbioru rozmytego A**.

Funkcja przynależności przypisuje każdemu elementowi x ∈ X jego **stopień przynależności do zbioru rozmytego A**. Można rozróżnić trzy przypadki:

- Stopień przynależności równy 0 oznacza, że element nie należy do zbioru rozmytego.
- Stopień przynależności równy 1 oznacza, że element w pełni należy do zbioru rozmytego.
- Stopień przynależności należący do przedziału (0, 1) oznacza, że element częściowo należy do zbioru rozmytego.

Za pomocą zbiorów rozmytych można formalnie określić pojęcia nieprecyzyjne, takie jak "średni wzrost", "wysoka temperatura", "duże miasto".

Źródła definiują pojęcie zbioru rozmytego i funkcji przynależności, a także wspominają o singletonach rozmytych i liczbach rozmytych (zbiorach rozmytych określonych w zbiorze liczb rzeczywistych spełniających pewne warunki). Nie zawierają jednak opisu czterech wybranych klas funkcji przynależności (takich jak np. funkcje trójkątne, trapezoidalne, Gaussa itp.), choć w sekcji o głębokim uczeniu przedstawione są wykresy funkcji aktywacji (sigmoidalna, ReLU, softplus, tanh), które mogą mieć kształty podobne do niektórych funkcji przynależności używanych w praktyce logiki rozmytej. Należy jednak zaznaczyć, że źródła nie łączą tych konkretnych funkcji z zastosowaniem jako funkcje przynależności w logice rozmytej.

## Omów architekturę i zastosowanie konwolucyjnych sieci neuronowych (CNN)

**Architektura i zastosowanie konwolucyjnych sieci neuronowych (CNN)**

Konwolucyjne sieci neuronowe (ang. CNN - Convolutional Neural Network) są rodzajem sieci neuronowych wykorzystywanych m.in. do analizy danych, w których istotne jest wewnętrzne powiązanie między wartościami, takie jak **obrazy**. W przypadku analizy obrazu, obraz nie może być traktowany jedynie jako wektor wartości pikseli, ponieważ **dużą rolę odgrywa sąsiedztwo pikseli**.

**Architektura CNN** charakteryzuje się **przestrzennie lokalnymi połączeniami**, przynajmniej w początkowych warstwach. Każda jednostka (neuron) w pierwszej ukrytej warstwie otrzymuje tylko niewielki podzbiór pikseli z małego, lokalnego regionu obrazka. Dzięki temu uwzględniane jest sąsiedztwo pikseli, przynajmniej lokalnie, a w przypadku kolejnych warstw również globalnie. Wagi w CNN układają się we wzorce replikowane w jednostkach każdej z warstw. Wzorzec wag, który powtarza się na przestrzeni wielu lokalnych regionów, nosi nazwę **jądra** (ang. kernel). Schemat łączenia jądra wag z pikselami (lub przestrzennie zorganizowanymi jednostkami w następnej warstwie) nazywa się **splotem lub konwolucją** (ang. convolution). Kluczową ideą warstwy konwolucyjnej jest zastosowanie wielu małych warstw liniowych (jąder) zamiast jednej dużej, gęstej warstwy liniowej z połączeniem między każdym wejściem a każdym wyjściem. Małe jądra (np. z mniej niż 25 danymi wejściowymi) są używane na każdej pozycji wejściowej obrazu. Przykłady jednowymiarowej i dwuwymiarowej operacji konwolucji są zilustrowane w źródłach.

Zaletą takiej architektury jest również **drastyczna redukcja liczby wag** w porównaniu do sieci w pełni połączonych, nawet o kilka rzędów wielkości.

Chociaż źródła nie wymieniają wprost szczegółowych zastosowań CNN poza analizą obrazów, konwolucyjne sieci neuronowe są powszechnie stosowane w zadaniach związanych z przetwarzaniem obrazu, takich jak:

- Klasyfikacja obrazów (np. rozpoznawanie obiektów) [129 - ogólnie dla systemów uczących się, CNN jest specyficznym typem]
- Detekcja i segmentacja obiektów
- Rozpoznawanie twarzy
- Analiza medyczna (np. analiza obrazów rentgenowskich)

Źródła wskazują na analizę obrazów jako główną motywację do zastosowania CNN.

## Omów metody selekcji osobników stosowane w tradycyjnym algorytmie genetycznym

**Metody selekcji osobników stosowane w tradycyjnym algorytmie genetycznym**

Algorytmy genetyczne są algorytmami przeszukiwania przestrzeni rozwiązań, opartymi na mechanizmach doboru naturalnego. Przetwarzają populację osobników, z których każdy jest propozycją rozwiązania problemu i ma przypisaną wartość funkcji przystosowania (fitness), określającą jakość rozwiązania. Aby zapewnić efektywne działanie algorytmu, **nowa populacja potomków powinna być tworzona na podstawie "najlepszych" osobników** z populacji rodzicielskiej. W tym celu stosuje się metody selekcji. Metoda selekcji powinna prowadzić do populacji potomnych o wyższej średniej wartości funkcji przystosowania.

Źródła wymieniają następujące metody selekcji stosowane w algorytmach genetycznych:

- **Metoda koła ruletki**
- **Selekcja rankingowa**
- **Selekcja turniejowa**
- Selekcja (μ + λ)

Omówienie wybranych metod:

- **Metoda koła ruletki:** Każdemu chromosomowi w populacji przypisuje się **wycinek koła ruletki proporcjonalny do wartości jego funkcji przystosowania**. Im większa wartość funkcji przystosowania, tym większy wycinek koła, co oznacza **większe prawdopodobieństwo wyboru** danego chromosomu do nowej populacji. Aby utworzyć nową populację, losuje się tyle razy, ile wynosi pożądana liczebność nowej populacji, wybierając chromosomy, na które wypadnie losowanie na kole ruletki.

  - **Wada:** Ewolucja z każdym krokiem zwalnia, a metoda **słabiej rozróżnia osobniki dobre od słabszych**, zwłaszcza gdy osobniki są do siebie podobne, a ich wartości funkcji przystosowania są zbliżone.

- **Selekcja rankingowa:** Osobniki w populacji rodzicielskiej są **sortowane według wartości funkcji przystosowania**, tworząc listę rankingową. Osobnik z największą wartością funkcji przystosowania uzyskuje najwyższą rangę. Przygotowana do osobnika ranga jest następnie argumentem funkcji R, która wyznacza współczynnik selekcji, będący podstawą do obliczenia prawdopodobieństwa jego wyboru do nowej populacji. Im wyższy współczynnik selekcji, tym większe prawdopodobieństwo wylosowania osobnika. Losowanie może odbywać się podobnie jak w metodzie koła ruletki, ale wielkość wycinka zależy od rangi i współczynnika selekcji.

  - **Wada:** Metoda jest **niewrażliwa na różnice** pomiędzy kolejnymi osobnikami na liście rankingowej. Może się zdarzyć, że sąsiadujące rozwiązania na liście mają różne wartości funkcji oceny, ale otrzymują podobną liczbę potomstwa.

- **Selekcja turniejowa:** Polega na tym, że osobniki z danej populacji "**rozgrywają między sobą turniej**". Najpierw ustala się rozmiar turnieju (zazwyczaj < 5). Do każdego pojedynku losowo wyznacza się ustaloną liczbę grup chromosomów. Zwycięzcy poszczególnych pojedynków rywalizują dalej do momentu wyłonienia zwycięzcy całego turnieju. Aby uzyskać odpowiednią liczebność nowej populacji, przeprowadza się odpowiednią liczbę turniejów. Źródło zawiera schemat ilustrujący działanie selekcji turniejowej dla rozmiaru turnieju 3.

Źródła wymieniają również selekcję (μ + λ), ale nie opisują jej działania szczegółowo.

## Podaj algorytm programowania genetycznego. Omów sposób reprezentacji programów w programowaniu genetycznym.

**Algorytm programowania genetycznego i sposób reprezentacji programów**

**Programowanie genetyczne (GP)** jest **rozszerzeniem klasycznego algorytmu genetycznego** i służy do **automatycznego generowania programów komputerowych**. W GP dokonuje się ewolucji programów w celu znalezienia programu optymalnego dla danego zadania. Populacja programów podlega działaniu odpowiednich operatorów genetycznych.

**Sposób reprezentacji programów w programowaniu genetycznym:**
John Koza zaproponował **przedstawienie programów w postaci hierarchicznej, drzewiastej struktury**. W związku z tym, w programowaniu genetycznym chromosomy kodowane są jako **drzewa składające się z węzłów i krawędzi**. Podstawowa informacja o programie zawarta jest w węzłach, a krawędzie określają relacje między nimi. Drzewo reprezentujące program posiada węzły różnego typu:

- **Korzeń:** Najwyższy węzeł drzewa.
- **Węzły pośrednie:** Reprezentują funkcje (np. operacje arytmetyczne, logiczne, programowe).
- **Węzły terminalne:** Reprezentują wartości końcowe, takie jak zmienne, stałe, sensory systemu.

Przykład drzewa dla wyrażenia `a+2*b*c` ilustruje tę strukturę.

W przeciwieństwie do konwencjonalnych algorytmów genetycznych operujących na łańcuchach o stałej długości (co nie jest odpowiednie do kodowania programów o zmiennej strukturze i rozmiarze), w programowaniu genetycznym **rozmiar, forma i zawartość struktur reprezentujących programy mogą zmieniać się dynamicznie w czasie ewolucji**. Zbiór możliwych struktur jest tworzony rekurencyjnie ze zdefiniowanego zbioru funkcji F i zbioru terminali T.

**Algorytm (lub proces) programowania genetycznego:**
Proces programowania genetycznego rozpoczyna się od **utworzenia w sposób losowy początkowej populacji programów** (drzew) złożonych z funkcji i terminali właściwych dla danego problemu. Następnie, dla każdego programu w populacji, **oceniany jest jego stopień realizacji postawionego zadania** za pomocą miary dopasowania (fitness measure). Ocena ta jest zazwyczaj przeprowadzana przez **uruchomienie każdego programu dla grupy przypadków testowych** (fitness cases). Miara dopasowania może być np. sumą lub średnią wyników dla przypadków testowych, liczbą zdobytych punktów, liczbą prawidłowo rozpoznanych wzorców lub minimalizacją błędów.

Nowa populacja tworzona jest na podstawie zasady przeżycia i reprodukcji najlepszych osobników oraz stosowania operatorów genetycznych. Podstawowe operatory to **reprodukcja i krzyżowanie**.

- **Reprodukcja** polega na wyborze osobnika (programu) i skopiowaniu go (bez zmian) do nowej populacji, często stosując selekcję proporcjonalną do dopasowania lub inne metody selekcji.
- **Krzyżowanie** polega na wyborze dwóch osobników (rodziców) na podstawie ich dopasowania. Następnie w każdym z rodziców losowo wybierany jest punkt krzyżowania. Fragmentem używanym przy krzyżowaniu jest poddrzewo o korzeniu w punkcie krzyżowania. Nowi potomkowie są tworzeni przez wymianę tych poddrzew między rodzicami.

Proces tworzenia nowych pokoleń (ocena, selekcja, krzyżowanie, reprodukcja) jest powtarzany. Algorytm kończy się zazwyczaj po wykonaniu określonej liczby przebiegów (pokoleń) lub po spełnieniu określonego warunku przez najlepsze rozwiązanie.

Oprócz reprodukcji i krzyżowania, w programowaniu genetycznym stosuje się operacje pomocnicze, takie jak mutacja, permutacja, edycja, enkapsulacja czy dziesiątkowanie.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/selfheavenly)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/selfheavenly)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
