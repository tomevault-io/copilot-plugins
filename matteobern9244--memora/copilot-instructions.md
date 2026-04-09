## memora

> > Integra `AGENTS.md` (conductor vincolante). Leggere sempre AGENTS.md prima.

# CLAUDE.md — Istruzioni per Claude Code

> Integra `AGENTS.md` (conductor vincolante). Leggere sempre AGENTS.md prima.

## Contesto

- **Progetto**: Memora — C# / .NET 10 / ASP.NET Core + React/Vite + React Native
- **Lingua comunicazione**: Italiano — **Lingua codice**: Inglese
- **Solution**: `Memora.sln` con progetti `src/Memora.Api`, `src/Memora.Domain`, `src/Memora.Infrastructure`
- **Test projects**: `tests/Memora.Domain.Tests`, `tests/Memora.Infrastructure.Tests`, `tests/Memora.Api.Tests`

## Regole Claude Code

### Prima di ogni task

1. Leggi `AGENTS.md` e `PROJECT.md`
2. Riformula l'obiettivo in italiano
3. Ispeziona codice esistente con gli strumenti Read/Grep/Glob
4. Se ambiguo, chiedi — non inventare

### Comandi

```bash
dotnet test                                          # Tutti i test
dotnet test --collect:"XPlat Code Coverage"          # Con coverage
dotnet build --no-restore                            # Build
dotnet run --project src/Memora.Api                  # Dev server
cd frontend/web && npm run test                      # Test frontend
cd frontend/web && npm run build                     # Build frontend
```

### Formato output

```
## Piano
- Obiettivo: [1-2 righe]
- File impattati: [lista]
- Principi: [Clean Architecture, MediatR, FluentValidation, TDD, etc.]
- TDD: RED → GREEN → REFACTOR
- Criteri accettazione: [lista]
- Regressioni: [lista check]

## Implementazione
[codice]

## Report
- File toccati: [lista]
- Comandi eseguiti: [lista]
- Coverage: [risultato]
- Rischi residui: [nessuno | lista]
```

### Stile C #

- Nullable reference types abilitati
- `IOptions<T>` per configurazione, mai stringhe hardcoded
- Constructor injection, nessun service locator
- Async/await per I/O, `CancellationToken` propagato
- Record per DTO/value objects, class per entità con identità
- Naming: PascalCase (metodi, proprietà, classi), camelCase (parametri, locali)
- **MediatR**: controller → `IMediator.Send()` → handler → domain service
- **FluentValidation**: ogni request DTO ha il suo validatore, integrato in `ValidationBehavior`
- **Clean Architecture**: `Memora.Domain` non dipende da `Infrastructure` o `Api`

### Priorità Claude Code

Architettura, TDD completo, refactoring complessi, debugging multi-file, code review.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matteobern9244)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/matteobern9244)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
