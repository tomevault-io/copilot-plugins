## 015-cursor-rules-cost-optimization

> - **Modyfikacja/dodawanie reguł w `.cursor/rules/*.mdc`** musi być zoptymalizowane pod kątem oszczędzania kosztów i unikania uruchamiania kosztownych flow agentów.


# Optymalizacja kosztów dla modyfikacji/dodawania reguł Cursor

## Zasady ogólne

- **Modyfikacja/dodawanie reguł w `.cursor/rules/*.mdc`** musi być zoptymalizowane pod kątem oszczędzania kosztów i unikania uruchamiania kosztownych flow agentów.

## Zasady szczegółowe

### 1. Minimalizacja kontekstu

- **Nie czytaj całych plików reguł** - używaj `grep` lub `read_file` z `offset`/`limit` do lokalizacji konkretnych sekcji.
- **Nie analizuj wszystkich reguł naraz** - modyfikuj tylko konkretną regułę/sekcję wskazaną przez użytkownika.
- **Unikaj szerokich wyszukiwań semantycznych** - używaj `grep` dla znanych nazw plików/sekcji.

### 2. Precyzyjne edycje

- **Używaj `search_replace`** z dokładnym `old_string` zamiast przepisywania całych plików.
- **Dodawaj tylko niezbędne sekcje** - nie przepisuj całych plików jeśli zmiana dotyczy jednej sekcji.
- **Sprawdzaj kontekst przed edycją** - używaj `read_file` z `offset`/`limit` aby zobaczyć tylko potrzebny fragment.

### 3. Unikanie kosztownych operacji

- **Nie uruchamiaj testów** przy modyfikacji reguł (reguły nie są kodem produkcyjnym).
- **Nie analizuj całego codebase** przy dodawaniu reguły - skup się tylko na konkretnym pliku reguły.
- **Nie wykonuj pełnych analiz semantycznych** - używaj precyzyjnych narzędzi (`grep`, `read_file` z limitami).

### 4. Workflow dla modyfikacji reguł

1. **Lokalizacja:** Użyj `grep` lub `read_file` z `offset`/`limit` do znalezienia sekcji.
2. **Edycja:** Użyj `search_replace` z dokładnym kontekstem.
3. **Weryfikacja:** Sprawdź tylko zmieniony fragment (`read_file` z limitami), nie cały plik.
4. **Zakończenie:** Nie uruchamiaj testów, nie analizuj całego projektu.

### 5. Przykłady dobrej praktyki

✅ **DOBRZE:**
```bash
# 1. Znajdź sekcję
grep -n "Feature Flags" docs/cursor-rules/pl/coding-standards.mdc

# 2. Przeczytaj tylko potrzebny fragment
read_file docs/cursor-rules/pl/coding-standards.mdc offset=230 limit=20

# 3. Edytuj precyzyjnie
search_replace file_path=... old_string="## Feature Flags\n\n..." new_string="..."
```

❌ **ŹLE:**
```bash
# 1. Czytanie całego pliku
read_file docs/cursor-rules/pl/coding-standards.mdc

# 2. Szeroka analiza semantyczna
codebase_search query="How are feature flags used in cursor rules?"

# 3. Uruchamianie testów
run_terminal_cmd "php artisan test"
```

## Wyjątki

- Jeśli użytkownik **explicitnie** poprosi o analizę wszystkich reguł lub uruchomienie testów - wtedy można to zrobić.
- Jeśli modyfikacja reguły wymaga sprawdzenia wpływu na kod produkcyjny - użyj precyzyjnych narzędzi, nie pełnej analizy.

---

**Cel:** Zmniejszenie kosztów tokenów przy modyfikacji reguł Cursor poprzez precyzyjne, zlokalizowane operacje zamiast szerokich analiz.

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
