## minecraftplugin

> Stworzenie wysoce skalowalnego szablonu (template) pluginu Minecraft (Paper API). Plugin oferuje kasyno, gdzie walutą jest złoto pobierane z ekwipunku i Enderchesta gracza. Całość obsługiwana jest przez Chest GUI wywoływane jedną główną komendą. Szablon jest przygotowany dla 4-osobowego zespołu deweloperskiego i generuje wyłącznie plik .jar (standalone plugin).

# Projekt: Minecraft Casino Plugin - Dokumentacja i Standardy

## 1. Cel Projektu
Stworzenie wysoce skalowalnego szablonu (template) pluginu Minecraft (Paper API). Plugin oferuje kasyno, gdzie walutą jest złoto pobierane z ekwipunku i Enderchesta gracza. Całość obsługiwana jest przez Chest GUI wywoływane jedną główną komendą. Szablon jest przygotowany dla 4-osobowego zespołu deweloperskiego i generuje wyłącznie plik .jar (standalone plugin).

## 2. Stos Technologiczny
* Język: Java 21+
* API: Paper API
* System budowania: Gradle (KTS - Kotlin DSL preferowane dla czytelności)
* Testy: JUnit 5, MockBukkit (do testów integracyjnych i logiki Bukkit API)
* CI/CD: GitHub Actions (budowanie, testowanie, lintowanie)

## 3. Architektura i Wzorce Projektowe
Projekt musi ściśle przestrzegać zasad SOLID. Monolityczne klasy są zabronione. Wymagana jest separacja warstw:

* **core:** Główna klasa pluginu, menedżerowie cyklu życia, wstrzykiwanie zależności.
* **economy:** Logika zliczania, zabierania i dodawania złota (Enderchest + Inventory). Traktowane jako niezależny serwis (`EconomyManager`).
* **gui:** Abstrakcyjny framework do tworzenia Chest GUI (np. wzorzec Builder). Żadna gra nie tworzy inwentarza bezpośrednio, lecz korzysta z menedżera GUI.
* **commands:** Oddzielne klasy dla komend. Jedna główna komenda otwierająca menu główne.
* **games:** Warstwa abstrakcji. Definicja interfejsu `CasinoGame` (metody: `start()`, `stop()`, `getIcon()`, `getName()`). Każda gra to osobny pakiet/moduł implementujący ten interfejs.
* **listeners:** Nasłuchiwacze zdarzeń, rejestrowane dynamicznie, z izolacją logiki od samej obsługi zdarzenia.

## 4. Wytyczne dla Agenta (Claude)
* Repozytorium to wyłącznie kod pluginu. Nie twórz żadnych skryptów startowych serwera (start.sh, .bat) ani konfiguracji serwerowych.
* System budowania ma generować czysty plik `.jar` gotowy do wrzucenia do folderu `plugins`.
* Przy generowaniu kodu zawsze stosuj anglojęzyczne nazewnictwo zmiennych, klas i metod.
* Każdy nowy komponent musi być poparty interfejsem.
* Skup się na stworzeniu struktury katalogów i szkieletu, w którym każdy z 4 deweloperów może dodać nową grę bez modyfikowania głównego kodu pluginu.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maciekgangus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
