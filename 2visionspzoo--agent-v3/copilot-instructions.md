## agent-v3

> **OBJAW:** `UnicodeEncodeError: 'charmap' codec can't encode character`

# 📋 INSTRUKCJE DLA CLAUDE - NAJWAŻNIEJSZE ZASADY PRACY NAD PROJEKTEM AGENT.v3

## ⛔ NAJCZĘSTSZE BŁĘDY DO UNIKANIA

### 1. ❌ **PROBLEM: Używanie emoji i Unicode w kodzie Python**
**OBJAW:** `UnicodeEncodeError: 'charmap' codec can't encode character`

**✅ ROZWIĄZANIE:** 
```python
# NIGDY NIE UŻYWAJ:
print("✅ Sukces")
print("🔧 Naprawa")

# ZAWSZE UŻYWAJ:
print("[OK] Sukces")
print("[TOOL] Naprawa")
print("[ERROR] Błąd")
print("[INFO] Informacja")
```

**ZAWSZE DODAWAJ NAGŁÓWEK DO KAŻDEGO SKRYPTU PYTHON:**
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import sys
import io
# KRYTYCZNE dla Windows:
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
sys.stderr = io.TextIOWrapper(sys.stderr.buffer, encoding='utf-8')
```

---

### 2. ❌ **PROBLEM: Próby uruchamiania Pythona w analysis tool (REPL)**
**OBJAW:** Claude próbuje używać `invoke("repl")` dla kodu Python

**✅ ROZWIĄZANIE:**
- **NIE UŻYWAJ** analysis tool (REPL) dla Pythona - obsługuje TYLKO JavaScript!
- **ZAWSZE** twórz skrypty testowe w `C:\agent.v3\skrypty_testowe\`
- **INFORMUJ** użytkownika: "Utworzyłem skrypt: test_xyz.py. Uruchom: `python skrypty_testowe/test_xyz.py`"

---

### 3. ❌ **PROBLEM: Błędny dostęp do ConfigManager**
**OBJAW:** `AttributeError: 'ConfigManager' object has no attribute 'config'`

**✅ ROZWIĄZANIE:**
```python
# ŹLE:
config = config_manager.config

# DOBRZE:
config = config_manager.get_config()
```

---

### 4. ❌ **PROBLEM: Usuwanie działających funkcjonalności**
**OBJAW:** Działający kod przestaje działać po "poprawkach"

**✅ ROZWIĄZANIE:**
- **NIGDY** nie usuwaj istniejącego kodu bez wyraźnej prośby
- **ZAWSZE** rozszerzaj funkcjonalność, nie zastępuj
- **PRZED ZMIANĄ** sprawdź czy kod działa: "Czy mogę zmodyfikować funkcję X?"
- **ZACHOWUJ** kompatybilność wsteczną

---

### 5. ❌ **PROBLEM: Nieprawidłowe ścieżki Windows**
**OBJAW:** Błędy dostępu do plików

**✅ ROZWIĄZANIE:**
```python
from pathlib import Path

# DOBRZE:
file_path = Path("C:/agent.v3/data/file.txt")
# lub
file_path = Path(r"C:\agent.v3\data\file.txt")

# ŹLE:
file_path = "C:\agent.v3\data\file.txt"  # BŁĄD - \a \d to escape sequences!
```

---

### 6. ❌ **PROBLEM: Zmiany bez testowania**
**OBJAW:** Wprowadzanie zmian które psują system

**✅ ROZWIĄZANIE - WORKFLOW:**
1. **NAJPIERW** sprawdź obecny stan: "Sprawdzę czy funkcja działa..."
2. **POTEM** zaproponuj zmianę: "Proponuję następującą modyfikację..."
3. **ZAPYTAJ** o zgodę: "Czy mogę wprowadzić te zmiany?"
4. **STWÓRZ** skrypt testowy przed zmianą
5. **POINFORMUJ** o testach: "Uruchom test_xyz.py aby sprawdzić zmiany"

---

### 7. ❌ **PROBLEM: Przekraczanie limitów odpowiedzi**
**OBJAW:** Zbyt wiele zmian naraz, niepełne odpowiedzi

**✅ ROZWIĄZANIE:**
- Dziel zmiany na małe kroki (max 2-3 pliki na raz)
- Twórz prompt kontynuacji: "Zapisałem stan w prompt_kontynuacji.md"
- Priorytetyzuj: "Najpierw naprawię krytyczny błąd X, potem zajmę się Y"

---

### 8. ❌ **PROBLEM: Błędy JSON/JSONL**
**OBJAW:** `json.decoder.JSONDecodeError`, `TypeError: Object of type int64 is not JSON serializable`

**✅ ROZWIĄZANIE:**
```python
# Zawsze używaj konwertera dla numpy types:
def numpy_to_python(obj):
    """Konwertuj numpy types do Python types"""
    import numpy as np
    if isinstance(obj, np.integer):
        return int(obj)
    elif isinstance(obj, np.floating):
        return float(obj)
    elif isinstance(obj, np.bool_):
        return bool(obj)
    elif isinstance(obj, np.ndarray):
        return obj.tolist()
    elif isinstance(obj, dict):
        return {k: numpy_to_python(v) for k, v in obj.items()}
    elif isinstance(obj, list):
        return [numpy_to_python(item) for item in obj]
    return obj

# Przed zapisem JSON:
clean_data = numpy_to_python(data)
json.dump(clean_data, f, ensure_ascii=False, indent=2)
```

---

### 9. ❌ **PROBLEM: Brak skryptów testowych**
**OBJAW:** Zmiany wprowadzane bez możliwości weryfikacji

**✅ ROZWIĄZANIE - SZABLON TESTU:**
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Test dla zmian w module X
Data: [data]
Cel: [opis co testujemy]
"""

import sys
import os
sys.path.append(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))

def test_feature():
    print("[TEST] Testowanie funkcji X...")
    try:
        # Kod testu
        print("[OK] Test przeszedł")
        return True
    except Exception as e:
        print(f"[ERROR] Test nie powiódł się: {e}")
        return False

if __name__ == "__main__":
    success = test_feature()
    sys.exit(0 if success else 1)
```

---

### 10. ❌ **PROBLEM: Błędna integracja modułów**
**OBJAW:** Moduły nie współpracują poprawnie po zmianach

**✅ ROZWIĄZANIE:**
- Zawsze sprawdzaj zależności: "Ten moduł używa X, sprawdzę czy moja zmiana nie zepsuje X"
- Dokumentuj integracje: "Moduł A wywołuje B.metoda() która zwraca..."
- Testuj przepływ danych: "Dane płyną z A → B → C, sprawdzę każdy krok"

---

## 🎯 ZŁOTE ZASADY PRACY

### 1. **ZASADA MAŁYCH KROKÓW**
- Jedna zmiana = jeden problem
- Max 3 pliki na raz
- Zawsze test po każdej zmianie

### 2. **ZASADA KOMUNIKACJI**
```
1. "Znalazłem problem X..."
2. "Proponuję rozwiązanie Y..."
3. "Czy mogę wprowadzić zmiany?"
4. "Utworzyłem test Z..."
5. "Uruchom komendę..."
```

### 3. **ZASADA DOKUMENTACJI**
- Każda zmiana = komentarz w kodzie
- Każda sesja = prompt_kontynuacji.md
- Każdy test = opis co i dlaczego testujemy

### 4. **ZASADA BEZPIECZEŃSTWA**
- Backup przed zmianą (jeśli plik > 1000 linii)
- Nigdy nie usuwaj - komentuj
- Zawsze zachowuj kompatybilność

---

## 📁 STRUKTURA PROJEKTU - KLUCZOWE LOKALIZACJE

```
C:\agent.v3\
├── app/                    # Główna aplikacja
│   ├── main.py            # Punkt wejścia
│   ├── config.py          # Konfiguracja
│   ├── modules/           # Moduły analizy
│   ├── patterns/          # Wykrywanie wzorców
│   └── core/              # Rdzeń systemu
├── config/                 # Pliki konfiguracyjne
│   └── .env               # Zmienne środowiskowe
├── data/                  # Dane i wyniki
├── logi/                  # Logi aplikacji
└── skrypty_testowe/       # TUTAJ TWÓRZ WSZYSTKIE TESTY!
```

---

## 🔄 PRZYKŁADOWY WORKFLOW NAPRAWY

```python
# 1. DIAGNOZA
print("[INFO] Sprawdzam problem z modułem X...")
# Czytaj pliki, analizuj logi

# 2. PROPOZYCJA
print("[INFO] Znalazłem błąd w linii 123...")
print("[INFO] Proponuję następującą naprawę...")

# 3. PYTANIE
print("[?] Czy mogę wprowadzić tę zmianę?")

# 4. IMPLEMENTACJA (po zgodzie)
print("[INFO] Wprowadzam zmiany...")
# Edytuj pliki

# 5. TEST
print("[INFO] Tworzę skrypt testowy...")
# Utwórz test w skrypty_testowe/

# 6. INSTRUKCJA
print("[OK] Zmiany wprowadzone")
print("[RUN] python skrypty_testowe/test_fix_x.py")

# 7. DOKUMENTACJA
print("[INFO] Prompt kontynuacji: prompt_kontynuacji_[data].md")
```

---

## ⚠️ PAMIĘTAJ

1. **Windows + Python = problemy z encoding** - zawsze ustaw UTF-8
2. **Analysis tool ≠ Python** - tylko JavaScript!
3. **Małe kroki = mniej błędów**
4. **Testy = pewność że działa**
5. **Dokumentacja = łatwa kontynuacja**

---

*Te instrukcje powstały na podstawie analizy 50+ czatów z projektem agent3*
*Stosując je, unikniesz 90% najczęstszych problemów*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2visionspzoo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/2visionspzoo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
