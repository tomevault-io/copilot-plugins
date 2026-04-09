## aromagic-landing

> - Odpowiadaj po polsku, chyba że user pisze po angielsku

# Aromagic — instrukcje globalne dla Claude

## Ogólne zasady
- Odpowiadaj po polsku, chyba że user pisze po angielsku
- User ma ADHD — krótkie odpowiedzi, nie zadawaj zbędnych pytań
- Nie dodawaj emoji, chyba że user poprosi

## Linear Workflow
Gdy user zgłasza buga, zmianę lub nową funkcję:
1. **Analizuj** co user ma na myśli
2. **Kategoryzuj**: Bug, Feature, Improvement lub Pomysł
3. **Ustaw priorytet**: Urgent / High / Medium / Low — zapytaj jeśli nie wiadomo
4. **Zapytaj** czy przypisać do kogoś (assignee)
5. **Stwórz issue** w Linear (po polsku, team: Aromagic, projekt: Aromagic)
6. **Pokaż** link i krótkie podsumowanie

- **Pomysły:** label `Pomysł`, status Backlog, priorytet None
- **QA:** fix działa → user przenosi do Zweryfikowane; nie działa → user wraca do In Progress + komentarz
- Zawsze przypisuj nowe issues do projektu Aromagic

## DECISIONS.md
- Plik: `DECISIONS.md` (w tym katalogu)
- Aktualizuj przy decyzjach architektonicznych/technicznych — nowa biblioteka, zmiana wzorca, wybór technologii, zmiana struktury danych
- NIE aktualizuj przy drobnych fixach, refaktorach bez zmiany podejścia, zmianach CSS
- Format: `- DATA [AUTOR]: decyzja — dlaczego (ARO-numer jeśli jest)`
- Autor: `Emilia`, `Claude`, lub `Emilia+Claude`
- Nowe wpisy na górze sekcji "Bieżące decyzje"

## Repozytoria
| Repo | Stack | Port |
|------|-------|------|
| aromagic-backend | Spring Boot, PostgreSQL, Redis, Keycloak | 8080 |
| aromagic-frontend | React 18, Vite, MUI | 3000 |
| aroma-stats | Spring Boot, Feign, Resilience4j | 8085 |
| aroma-pricing | Spring Boot, Caffeine, JSoup | 8082 |
| aroma-team | Spring Boot | 8084 |
| aroma-share | Spring Boot | 8083 |
| aroma-browser-plugin | TypeScript, CRXJS, Manifest v3/v2 | - |
| aromagic-admin-console-backend | Spring Boot, Feign, own JWT | 8100 |

## Wzorce kodu
- Backend: Controller → Service → Repository, Facade pattern, package-private, DTO na granicach
- Frontend: Context API, custom hooks, serwisy w `/services`, SOLID
- Plugin: strict TS, zero `any`, Manifest v3/v2

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emilia-chodorowska)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/emilia-chodorowska)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
