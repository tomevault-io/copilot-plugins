## blender-assist

> Zbiór twardych danych technicznych dotyczących środowiska Blender 5.1+, protokołu mostka TCP oraz obsługi operacji na obiektach 3D.

# Baza Wiedzy Domenowej Blender MCP (SSOT)

Zbiór twardych danych technicznych dotyczących środowiska Blender 5.1+, protokołu mostka TCP oraz obsługi operacji na obiektach 3D.

## 1. Mostek Komunikacyjny i Ograniczenia Konkurencji
- **Infrastruktura:** Architektura opiera się na serwerze FastMCP komunikującym się z jednoplikowym add-onem Blendera po protokole TCP.
- **Parametry połączenia:** Komunikacja odbywa się na `localhost` poprzez port `8765` (`127.0.0.1:8765`).
- **Pętla Główna (Main Thread):** Wszystkie wywołania API Blendera (`bpy.*`) MUSZĄ być wykonywane synchronicznie w głównym wątku Blendera. Wykorzystujemy do tego dedykowaną kolejkę operacji (addon queue). Zakaz wywoływania `bpy` w wątkach pobocznych.

## 2. Manipulacja Sceną i Renderowanie (Blender 5.1+)
- **Render i Packshot:** Wykorzystujemy renderowanie `Cycles` dla realistycznego oddania transmisji i absorpcji objętościowej (kamienie szlachetne). Użycie materiałów proceduralnych AI odbywa się przez `build_procedural_jewelry_material`.
- **Node Tools:** Operatory Node Tools rejestrują się poprzez identyfikatory w formacie `GeometryNodeTree.node_tool_idname`.
- **Odkrywanie Narzędzi (Advertise-and-Activate):** Agent NIGDY nie zgaduje parametrów dla generycznych operatorów ani narzędzi Node Tools. Przed użyciem `node_tool_invoke` należy ZAWSZE wywołać `get_blender_operator_schema`, aby pozyskać poprawny JSON schemat i argumenty RNA z Blendera.

## 3. Matryca Błędów (Error Codes)
Krytyczne statusy zwrotne mostka wymagające ścisłej obsługi błędów:
- `NON_MANIFOLD_MESH` – Skonwertowana/zewaluowana siatka posiada dziury, zero-powierzchniowe trójkąty lub niezamknięte wierzchołki (błąd dyskwalifikujący druk 3D).
- `COLLECTION_CONFLICT` – Konflikt w `bpy.data.collections`. Dana kolekcja istnieje w pamięci, ale nie jest przypięta do głównego drzewa bieżącej sceny.
- `BRIDGE_TIMEOUT` – Blender nie przetworzył zadania z kolejki przed upływem narzuconego limitu (operacje CAD: domyślnie 30s, render: max 120s).
- `SCRIPT_POLICY_VIOLATION` – Add-on zablokował podejrzany lub naruszający zasady jednowątkowości kod z wywołania `run_script`.

---
> Source: [EPIRjewelry/Blender_assist](https://github.com/EPIRjewelry/Blender_assist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
