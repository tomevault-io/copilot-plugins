## taran

> > Ten plik zawiera kluczowe wytyczne architektoniczne, konwencje i zasady inżynieryjne obowiązujące w projekcie **Taran** — wysokowydajnym narzędziu do testów obciążeniowych napisanym w Rust.

# CLAUDE.md — Instrukcje dla Claude Code

> Ten plik zawiera kluczowe wytyczne architektoniczne, konwencje i zasady inżynieryjne obowiązujące w projekcie **Taran** — wysokowydajnym narzędziu do testów obciążeniowych napisanym w Rust.

---

## Przegląd projektu

**Taran** to CLI-first narzędzie do generowania obciążenia HTTP/gRPC/WebSocket, alternatywa dla JMeter/K6.
Kluczowe cechy: zero GC, async (Tokio), skryptowanie Rhai, HDR Histogram, real-time TUI dashboard.

Szczegółowy plan realizacji: `docs/PLAN.md`

---

## Struktura workspace (Cargo workspace)

```
taran/
├── Cargo.toml            # workspace root
├── taran-cli/            # binarka — punkt wejścia, parsowanie CLI (clap)
├── taran-core/           # silnik wykonawczy: VU, scheduler, load profiles
├── taran-config/         # parsowanie scenariuszy TOML/YAML (serde)
├── taran-metrics/        # zbieranie metryk: HDR Histogram, lock-free counters
├── taran-report/         # generowanie raportów: JSON, CSV, HTML, TUI
├── taran-script/         # silnik skryptowy Rhai — API dla scenariuszy
├── taran-protocols/      # klienty protokołów: HTTP, gRPC, WebSocket, TCP
└── docs/                 # dokumentacja projektu
```

### Zależności między crate'ami (kierunek →  = "zależy od")

```
taran-cli → taran-core → taran-config
                       → taran-metrics
                       → taran-protocols
                       → taran-script
taran-report → taran-metrics
taran-script → taran-protocols
```

**Reguła:** Crate'y niższego poziomu (config, metrics, protocols) NIE mogą zależeć od crate'ów wyższego poziomu (core, cli). Zależności płyną od góry do dołu.

---

## Clean Architecture

Projekt stosuje zasady Clean Architecture zaadaptowane do Rust:

### Warstwy (od wewnętrznej do zewnętrznej)

1. **Domain (taran-core)** — czysta logika biznesowa
   - Definicje trait'ów (`Protocol`, `LoadProfile`, `MetricsCollector`, `ScriptEngine`)
   - Struktury domenowe (`VirtualUser`, `Scenario`, `Step`, `Assertion`, `TestResult`)
   - Brak zależności od frameworków I/O — operuje na trait'ach
   - Nie importuje `tokio`, `reqwest`, `tonic` bezpośrednio

2. **Application (taran-core::engine)** — orkiestracja
   - `TestRunner` — uruchamia scenariusze, zarządza lifecycle VU
   - `Scheduler` — implementuje load profiles (constant, ramp, spike)
   - Operuje na abstrakcjach (trait objects / generics), nie na konkretnych typach

3. **Infrastructure (taran-protocols, taran-metrics, taran-config, taran-script)** — implementacje
   - Konkretne implementacje trait'ów z warstwy Domain
   - `HttpClient` impl `Protocol`, `GrpcClient` impl `Protocol`
   - `HdrMetricsCollector` impl `MetricsCollector`
   - `RhaiScriptEngine` impl `ScriptEngine`
   - `TomlConfigLoader` impl `ConfigLoader`

4. **Presentation (taran-cli, taran-report)** — interfejs użytkownika
   - CLI parsing (clap), TUI dashboard (ratatui)
   - Raporty HTML/JSON/CSV
   - Dependency injection — łączy wszystkie warstwy

### Reguła zależności

```
Presentation → Application → Domain ← Infrastructure
```

Infrastructure zależy od Domain (implementuje trait'y), NIE odwrotnie.
Presentation łączy wszystko przez dependency injection (konstruktory).

---

## Zasady SOLID w Rust

### Single Responsibility Principle (SRP)

- Każdy crate ma jedną odpowiedzialność (patrz struktura workspace)
- Każdy moduł (`mod`) ma wyraźnie zdefiniowany zakres
- Struktury mają jedną odpowiedzialność:
  - `Scheduler` — tylko harmonogram uruchamiania VU
  - `HttpClient` — tylko komunikacja HTTP
  - `HdrCollector` — tylko zbieranie metryk
- **NIE** twórz "god structs" łączących wiele odpowiedzialności

### Open/Closed Principle (OCP)

- Nowe protokoły dodawane przez implementację trait `Protocol`, bez modyfikacji `taran-core`
- Nowe load profiles przez implementację trait `LoadProfile`
- Nowe formaty raportów przez implementację trait `Reporter`
- Używaj enum dispatch (`enum_dispatch` crate) lub trait objects (`Box<dyn Protocol>`) do polimorfizmu

```rust
// ✅ Otwarte na rozszerzenie
pub trait Protocol: Send + Sync {
    async fn execute(&self, request: &Request) -> Result<Response>;
    fn protocol_name(&self) -> &str;
}

// Nowy protokół = nowy struct + impl, zero zmian w istniejącym kodzie
pub struct GraphqlClient { /* ... */ }
impl Protocol for GraphqlClient { /* ... */ }
```

### Liskov Substitution Principle (LSP)

- Wszystkie implementacje trait'ów muszą spełniać kontrakt trait'a bez niespodzianek
- Jeśli trait definiuje `async fn execute() -> Result<Response>`, każda implementacja musi zwracać sensowny `Response` lub `Error` — nigdy `panic!()`
- Testy integracyjne powinny być parametryzowane po trait'ach, nie po konkretnych typach

### Interface Segregation Principle (ISP)

- Trait'y powinny być małe i fokusowane
- **NIE** twórz "fat traits" z 20 metodami
- Preferuj kompozycję trait'ów:

```rust
// ✅ Segregowane traity
pub trait Connectable {
    async fn connect(&mut self) -> Result<()>;
    async fn disconnect(&mut self) -> Result<()>;
}

pub trait RequestExecutor {
    async fn execute(&self, req: &Request) -> Result<Response>;
}

pub trait Protocol: Connectable + RequestExecutor + Send + Sync {}

// ✅ Blanket implementation
impl<T: Connectable + RequestExecutor + Send + Sync> Protocol for T {}
```

### Dependency Inversion Principle (DIP)

- `taran-core` definiuje trait'y (abstrakcje)
- `taran-protocols`, `taran-metrics` itd. implementują te trait'y
- `taran-cli` (composition root) łączy implementacje z abstrakcjami
- **NIGDY** nie importuj `taran-protocols` z `taran-core`

```rust
// taran-core/src/engine.rs
// ✅ Zależy od abstrakcji
pub struct TestRunner<P: Protocol, M: MetricsCollector> {
    protocol: P,
    metrics: M,
}

// taran-cli/src/main.rs
// ✅ Composition root — tu łączymy konkretne typy
let runner = TestRunner::new(
    HttpClient::new(config),
    HdrCollector::new(),
);
```

---

## DRY (Don't Repeat Yourself)

- Wspólna logika wyciągnięta do modułów utility w odpowiednim crate'cie
- Makra Rust (`macro_rules!`) do eliminacji boilerplate'u tam, gdzie to uzasadnione
- Derive macros (`serde::Serialize`, `Debug`, `Clone`) zamiast ręcznych implementacji
- Wspólne typy (`Duration`, `Url`, `HeaderMap`) re-eksportowane z jednego miejsca
- **NIE** duplikuj definicji błędów — jeden `Error` enum per crate z `thiserror`

```rust
// ✅ Jeden Error enum per crate
#[derive(Debug, thiserror::Error)]
pub enum ProtocolError {
    #[error("Connection failed: {0}")]
    ConnectionFailed(#[source] std::io::Error),
    #[error("Request timeout after {0:?}")]
    Timeout(Duration),
    #[error("Invalid response: {0}")]
    InvalidResponse(String),
}
```

---

## KISS (Keep It Simple, Stupid)

- Preferuj proste rozwiązania nad "sprytne"
- Nie zawsze potrzeba trait object — generics mogą wystarczyć (i są szybsze)
- `String` zamiast `Cow<'a, str>` dopóki profiling nie wykaże, że to bottleneck
- Proste `enum` zamiast skomplikowanych hierarchii typów
- Unikaj nadmiernego użycia makr — makra utrudniają debugowanie
- Nie implementuj feature'ów "na zapas" (patrz YAGNI)
- Preferuj `unwrap_or_default()` nad skomplikowane chain'y `map/and_then` gdy domyślna wartość jest oczywista

---

## YAGNI (You Aren't Gonna Need It)

- Implementuj TYLKO to, co jest potrzebne w bieżącej fazie (patrz `docs/PLAN.md`)
- Nie dodawaj wsparcia dla protokołu, zanim nie będzie potrzebny
- Nie optymalizuj przedwcześnie — najpierw poprawna implementacja, potem benchmark, potem optymalizacja
- Nie twórz abstrakcji "na wszelki wypadek" — abstrakcja powinna wynikać z realnej potrzeby (minimum 2 użycia)
- Pierwsza implementacja może być uproszczona (`todo!()` dla niekrytycznych ścieżek)

---

## CQRS (Command Query Responsibility Segregation)

Zastosowanie CQRS w kontekście Taran:

### Commands (zmieniają stan)

- `RunTest` — uruchom scenariusz obciążeniowy
- `StopTest` — zatrzymaj trwający test
- `RecordMetric` — zapisz pojedynczy pomiar
- `ScaleUp` / `ScaleDown` — zmień liczbę VU w trakcie testu

### Queries (odczytują stan, bez side effects)

- `GetCurrentMetrics` — pobierz bieżące metryki (snapshot)
- `GetTestStatus` — status testu (running, finished, error)
- `GetScenarioConfig` — odczytaj konfigurację scenariusza
- `GenerateReport` — wygeneruj raport z zebranych metryk

### Separacja w kodzie

```rust
// ✅ Oddzielone ścieżki zapisu i odczytu metryk
pub trait MetricsWriter: Send + Sync {
    fn record(&self, metric: Metric);
    fn record_latency(&self, step: &str, duration: Duration);
    fn increment_error(&self, step: &str, error_type: &str);
}

pub trait MetricsReader: Send + Sync {
    fn snapshot(&self) -> MetricsSnapshot;
    fn percentile(&self, step: &str, p: f64) -> Duration;
    fn throughput(&self) -> f64;
}

// Writer używany przez VU (hot path, lock-free)
// Reader używany przez TUI dashboard i reporter (cold path, może lock'ować)
```

---

## Separation of Concerns (SoC)

- **taran-cli:** TYLKO parsowanie argumentów i composition root. Zero logiki biznesowej.
- **taran-core:** TYLKO orkiestracja testów i definicje abstrakcji. Nie wie o HTTP/gRPC.
- **taran-protocols:** TYLKO implementacje protokołów. Nie wie o schedulerze.
- **taran-metrics:** TYLKO zbieranie i agregacja metryk. Nie wie o protokołach.
- **taran-config:** TYLKO parsowanie plików konfiguracyjnych. Zwraca domenowe struktury.
- **taran-script:** TYLKO silnik skryptowy. Eksponuje API, nie implementuje protokołów.
- **taran-report:** TYLKO generowanie raportów. Konsumuje `MetricsSnapshot`.

---

## Dependency Injection w Rust

Rust nie ma frameworka DI w stylu Spring/Autofac. Zamiast tego:

- **Constructor Injection:** Wszystkie zależności przekazywane przez konstruktory (`::new()`)
- **Generics / Trait bounds:** Statyczny dispatch (zero-cost) nad trait objects tam, gdzie wydajność jest krytyczna
- **Trait objects (`Box<dyn Trait>`):** Dynamiczny dispatch tam, gdzie potrzebna elastyczność runtime (np. pluginy, wybór protokołu z konfiguracji)
- **Composition Root:** `taran-cli/src/main.rs` — jedyne miejsce, gdzie tworzone są konkretne instancje i łączone z abstrakcjami
- **Builder pattern:** Dla struktur z wieloma opcjonalnymi parametrami

```rust
// ✅ Builder + constructor injection
let metrics = HdrCollector::builder()
    .with_precision(3)
    .with_max_value(Duration::from_secs(60))
    .build();

let runner = TestRunner::builder()
    .with_protocol(HttpClient::new(&config))
    .with_metrics(metrics)
    .with_scheduler(RampUpScheduler::new(100, Duration::from_secs(10)))
    .build()?;
```

---

## Error Handling

### Strategia

- **`thiserror`** w crate'ach bibliotecznych (taran-core, taran-protocols, ...): typowane enumeracje Error
- **`anyhow`** TYLKO w taran-cli (binarka): wrapping dowolnych błędów z kontekstem
- **NIGDY `unwrap()` / `expect()` w kodzie produkcyjnym** — dozwolone tylko w testach
- **`?` operator** do propagacji błędów w górę
- **Kontekstowe błędy:** Używaj `.context("msg")` / `.with_context(|| format!(...))` z `anyhow`

### Hierarchia błędów

```rust
// Każdy crate definiuje swój Error enum
// taran-core re-eksportuje Result type alias

pub type Result<T> = std::result::Result<T, TaranError>;

#[derive(Debug, thiserror::Error)]
pub enum TaranError {
    #[error("Configuration error: {0}")]
    Config(#[from] ConfigError),
    #[error("Protocol error: {0}")]
    Protocol(#[from] ProtocolError),
    #[error("Script error: {0}")]
    Script(#[from] ScriptError),
    #[error("Metrics error: {0}")]
    Metrics(#[from] MetricsError),
}
```

### Fail Fast Principle

- Waliduj konfigurację scenariusza PRZED rozpoczęciem testu (`taran validate`)
- Sprawdzaj dostępność endpointów na starcie (health check)
- Używaj `debug_assert!()` dla invariantów w kodzie
- Wczesny return (`guard clauses`) zamiast zagnieżdżonych `if/else`

---

## Konwencje kodowania Rust

### Nazewnictwo

| Element | Konwencja | Przykład |
|---|---|---|
| Crate | `snake_case` z prefixem `taran-` | `taran-core`, `taran-metrics` |
| Moduły | `snake_case` | `load_profile`, `virtual_user` |
| Struktury | `PascalCase` | `VirtualUser`, `TestRunner` |
| Trait'y | `PascalCase`, przymiotnikowe/rzeczownikowe | `Protocol`, `Connectable`, `MetricsCollector` |
| Funkcje/Metody | `snake_case` | `execute_request`, `record_latency` |
| Stałe | `SCREAMING_SNAKE_CASE` | `MAX_VU_COUNT`, `DEFAULT_TIMEOUT` |
| Typy generyczne | Pojedyncza wielka litera lub krótki `PascalCase` | `P: Protocol`, `M: MetricsCollector` |
| Feature flags | `snake_case` | `grpc_support`, `tui_dashboard` |

### Struktura plików w crate'cie

```
taran-core/
├── Cargo.toml
├── src/
│   ├── lib.rs              # re-eksporty publicznego API
│   ├── engine/
│   │   ├── mod.rs           # orkiestracja
│   │   ├── runner.rs        # TestRunner
│   │   └── scheduler.rs     # LoadProfile implementations
│   ├── model/
│   │   ├── mod.rs
│   │   ├── scenario.rs      # Scenario, Step, Assertion
│   │   ├── virtual_user.rs  # VirtualUser
│   │   └── request.rs       # Request, Response
│   ├── traits/
│   │   ├── mod.rs
│   │   ├── protocol.rs      # Protocol trait
│   │   ├── metrics.rs       # MetricsWriter, MetricsReader
│   │   └── script.rs        # ScriptEngine trait
│   └── error.rs             # TaranError, Result
└── tests/
    ├── integration/
    │   └── runner_test.rs
    └── common/
        └── mod.rs            # shared test utilities
```

### Formatowanie i linting

- `rustfmt` — obowiązkowe, konfiguracja w `rustfmt.toml`:
  ```toml
  max_width = 100
  edition = "2021"
  use_small_heuristics = "Max"
  tab_spaces = 4
  newline_style = "Unix"
  reorder_imports = true
  reorder_modules = true
  ```
  > **Uwaga:** `imports_granularity` i `group_imports` to unstable features dostępne
  > tylko na nightly rustfmt. Na stable toolchain (CI) są ignorowane, dlatego NIE
  > umieszczamy ich w `rustfmt.toml`.

- `clippy` — obowiązkowe, konfiguracja w `Cargo.toml` (`[workspace.lints.clippy]`):
  ```toml
  [workspace.lints.clippy]
  all = { level = "warn", priority = -1 }
  pedantic = { level = "warn", priority = -1 }
  nursery = { level = "warn", priority = -1 }
  unwrap_used = "deny"
  expect_used = "deny"
  panic = "deny"
  ```
  Grupy lintów (`all`, `pedantic`, `nursery`) muszą mieć `priority = -1`,
  aby indywidualne overridy (np. `missing_errors_doc = "allow"`) miały wyższy priorytet.

  Każdy crate musi zawierać `[lints] workspace = true` w swoim `Cargo.toml`.

- **Traktuj ostrzeżenia jako błędy w CI** (`cargo clippy -- -D warnings`)

---

## Async Rust — wytyczne

### Tokio Runtime

- Używaj `#[tokio::main]` TYLKO w `taran-cli/src/main.rs`
- Crate'y biblioteczne NIE uruchamiają runtime'u — przyjmują tokio context od callera
- Preferuj `tokio::spawn` dla niezależnych VU
- Używaj `tokio::select!` dla obsługi cancellation i timeoutów

### Concurrency

- **Hot path (metryki per request):** `AtomicU64`, `AtomicUsize` — zero locking
- **Agregacja metryk:** Thread-local `HdrHistogram` mergowany co 1s do globalnego
- **Współdzielony stan konfiguracji:** `Arc<Config>` — read-only po inicjalizacji
- **Mutable shared state:** `Arc<RwLock<T>>` tylko gdy absolutnie konieczne, preferuj channels
- **Channels:** `tokio::sync::mpsc` do komunikacji VU → MetricsCollector
- **NIGDY `std::sync::Mutex`** w async kodzie — używaj `tokio::sync::Mutex`

### Cancellation

- Każda długotrwała operacja musi respektować `CancellationToken` (`tokio-util`)
- Graceful shutdown: `Ctrl+C` → stop nowych VU → drain in-flight requests → finalize metrics
- Używaj `tokio::select!` z `token.cancelled()` branch'em

---

## Testowanie

### Strategia testowania

| Poziom | Narzędzie | Zakres |
|---|---|---|
| Unit tests | `#[cfg(test)]` + `mod tests` | Logika biznesowa, parsowanie, transformacje |
| Integration tests | `tests/` directory | Przepływ end-to-end w ramach crate'a |
| E2E tests | `taran-cli/tests/` | Uruchomienie binarki z prawdziwym scenariuszem |
| Mock server | `wiremock` | Mockowany HTTP server dla testów |
| Benchmarks | `criterion` | Wydajność krytycznych ścieżek |
| Property tests | `proptest` | Fuzzing parserów i transformacji |

### Konwencje testowe

```rust
// ✅ Nazewnictwo testów: should_<expected>_when_<condition>
#[test]
fn should_parse_duration_when_valid_string() { /* ... */ }

#[test]
fn should_return_error_when_url_is_invalid() { /* ... */ }

#[tokio::test]
async fn should_record_latency_when_request_completes() { /* ... */ }
```

- Każdy publiczny typ/trait musi mieć testy
- Testy async z `#[tokio::test]`
- Mock'owanie trait'ów z `mockall` crate
- Test fixtures w `tests/common/mod.rs` lub `tests/fixtures/`
- **Testy MUSZĄ przechodzić na CI przed merge'em**

---

## Performance — wytyczne krytyczne

Taran to narzędzie wydajnościowe — sam musi być wydajny. Każda decyzja projektowa musi uwzględniać wpływ na hot path.

### Hot path (per-request)

- **ZERO alokacji** na hot path (pre-allokuj bufory, reużywaj `String`/`Vec`)
- **ZERO locking** — atomics lub thread-local storage
- **ZERO syscalli** — batch'uj I/O (np. metryki flush co 1s, nie per request)
- Profiluj z `flamegraph` / `perf` / `cargo bench` przed i po zmianach

### Cold path (setup, reporting)

- Alokacje, locking, I/O — dozwolone
- Priorytet: czytelność kodu > wydajność

### Benchmarking

- `criterion` dla mikro-benchmarków
- Porównuj z baseline'em (K6, Goose) na tych samych scenariuszach
- CI benchmark regression checks (opcjonalnie)

---

## Dokumentacja

### Wymagania

- Każdy publiczny typ, trait, funkcja: `///` doc comment z przykładem
- `//!` na szczycie każdego modułu — opis odpowiedzialności
- `# Examples` w doc comments z `cargo test` runnable'ami
- `README.md` per crate z opisem i szybkim startem
- `CHANGELOG.md` w formacie Keep a Changelog

### Przykład

```rust
/// Scheduler odpowiedzialny za zarządzanie profilem obciążenia.
///
/// Kontroluje liczbę aktywnych Virtual Users w czasie trwania testu.
/// Wspiera różne profile: constant, ramp-up, stepped, spike.
///
/// # Examples
///
/// ```rust
/// use taran_core::engine::Scheduler;
/// use std::time::Duration;
///
/// let scheduler = Scheduler::ramp_up(
///     100,                          // target VU
///     Duration::from_secs(30),      // ramp-up time
///     Duration::from_secs(120),     // total duration
/// );
/// assert_eq!(scheduler.target_users(), 100);
/// ```
pub struct Scheduler { /* ... */ }
```

---

## Git — konwencje

### Conventional Commits

```
<type>(<scope>): <description>

feat(core): add ramp-up load profile scheduler
fix(protocols): handle connection timeout for HTTP/2
refactor(metrics): switch to thread-local HDR Histogram
docs(cli): add usage examples to README
test(core): add integration tests for VU lifecycle
perf(metrics): replace Mutex with AtomicU64 on hot path
ci: add Windows build to GitHub Actions
chore: update dependencies
```

### Typy commitów

| Typ | Opis |
|---|---|
| `feat` | Nowa funkcjonalność |
| `fix` | Naprawa błędu |
| `refactor` | Refaktoryzacja bez zmiany zachowania |
| `perf` | Optymalizacja wydajności |
| `test` | Dodanie/poprawienie testów |
| `docs` | Dokumentacja |
| `ci` | Zmiany w CI/CD |
| `chore` | Maintenance (deps, configs) |

### Branching

- `main` — zawsze stabilny, releasy tagowane wersją
- `feat/<name>` — nowe feature'y
- `fix/<name>` — bugfixy
- PR review wymagany przed merge'em do `main`

---

## Checklist przed każdym PR

- [ ] `cargo fmt --all --check` — formatowanie
- [ ] `cargo clippy --all-targets --all-features -- -D warnings` — linting
- [ ] `cargo test --all` — wszystkie testy przechodzą
- [ ] `cargo doc --no-deps` — dokumentacja się generuje
- [ ] Nowy publiczny API ma doc comments z przykładami
- [ ] Nowy kod ma testy (unit i/lub integration)
- [ ] Brak `unwrap()` / `expect()` w kodzie produkcyjnym
- [ ] Commit messages w formacie Conventional Commits
- [ ] CHANGELOG.md zaktualizowany (jeśli dotyczy)

---

## Kluczowe crate'y — referencja

| Crate | Wersja* | Przeznaczenie |
|---|---|---|
| `tokio` | latest | Async runtime |
| `reqwest` | latest | HTTP client (high-level) |
| `hyper` | latest | HTTP (low-level, HTTP/2) |
| `tonic` | latest | gRPC client |
| `tokio-tungstenite` | latest | WebSocket |
| `clap` | 4.x | CLI argument parsing (derive) |
| `serde` | 1.x | Serialization/deserialization |
| `toml` | latest | TOML parsing |
| `rhai` | latest | Embedded scripting |
| `hdrhistogram` | latest | HDR Histogram for latency |
| `ratatui` | latest | Terminal UI dashboard |
| `tracing` | latest | Structured logging |
| `thiserror` | latest | Error derive macros (libraries) |
| `anyhow` | latest | Error handling (binary) |
| `rustls` | latest | TLS (pure Rust) |
| `jsonpath-rust` | latest | JSONPath extraction |
| `regex` | latest | Regex extraction |
| `minijinja` | latest | HTML report templates |
| `criterion` | latest | Benchmarking |
| `wiremock` | latest | HTTP mock server (tests) |
| `mockall` | latest | Trait mocking (tests) |
| `proptest` | latest | Property-based testing |
| `tokio-util` | latest | CancellationToken, extras |

*Zawsze używaj najnowszych stabilnych wersji. Pinuj wersje w `Cargo.toml`.

---

## Typowe pułapki do unikania

1. **`std::sync::Mutex` w async** — użyj `tokio::sync::Mutex` lub lepiej atomics/channels
2. **`.clone()` na hot path** — profil przed klonowaniem, rozważ `Arc` lub referencje
3. **`String` zamiast `&str` w sygnaturach** — akceptuj `impl AsRef<str>` lub `&str` gdzie możliwe
4. **Fat trait objects** — segreguj trait'y, nie rób `Box<dyn Everything>`
5. **Brak `Send + Sync` bounds** — pamiętaj o nich w trait'ach używanych w async
6. **`panic!()` w bibliotece** — zwracaj `Result`, nigdy nie panikuj
7. **Blokujące I/O w async** — `tokio::task::spawn_blocking()` dla operacji blokujących
8. **Coordinated omission** — mierz czas od ZAPLANOWANIA żądania, nie od wysłania
9. **Brak graceful shutdown** — zawsze obsługuj `Ctrl+C` → drain → cleanup
10. **Nadmierny logging na hot path** — `tracing` z filtrami, TRACE/DEBUG wyłączone w benchmarkach

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shaqal7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
