## net-c-clean-architecture-skill

> |


# Clean Naming & Architecture Rules

You write code that humans read out loud. Every name you choose must survive this test:
**Can you say it in a sentence to a colleague without needing to explain what it "actually" means?**

If the answer is "well, what it *actually* does is..." — the name is wrong. Fix it before moving on.


## The Core Principle

Names reveal **intent**, never implementation. The reader should understand *what* something
does and *why* it exists without opening the file. Implementation details belong inside the
file, not on the label.


## Naming Rules

### 1. The Conversation Test

Before committing any name, speak this sentence out loud:

> "The [name] handles/returns/contains [what you think it does]."

If the sentence sounds unnatural or requires a footnote, rename it.

**Good:**
- "The `PatientRepository` handles persistence of patients." Natural.
- "The `InboundMapper` converts external messages to our domain model." Clear.
- "The `WorkflowEngine` orchestrates workflow execution." Obvious.

**Bad:**
- "The `DataHelper` handles... well, various data things." Too vague.
- "The `ProcessManager2` handles... the second version of process management?" Versioned names are never OK.
- "The `SqlPatientStore` handles patient storage in SQL." Implementation leaked into an abstraction name.

### 2. Layer-Appropriate Vocabulary

Each layer speaks its own language. Never mix vocabularies across boundaries.

**Domain layer** — speaks business language:
- `Patient`, `LabResult`, `Workflow`, `WorkflowStep`
- `IPatientRepository`, `IMessageQueue`, `IWorkflowEngine`
- Verbs: `Submit`, `Approve`, `Process`, `Complete`, `Cancel`

**Application layer** — speaks use-case language:
- `ProcessInboundMessage`, `OrchestrateWorkflow`, `SubmitLabResult`
- `InboundMessageHandler`, `WorkflowOrchestrator`
- Verbs: `Handle`, `Orchestrate`, `Execute`, `Coordinate`

**Infrastructure layer** — speaks technology language:
- `PostgresPatientRepository`, `RabbitMqMessageQueue`, `HttpLabClient`
- `SerilogLogger`, `PostgresSkipLockedQueue`, `SmtpEmailSender`
- Verbs: `Query`, `Insert`, `Serialize`, `Post`, `Connect`

**Presentation/API layer** — speaks HTTP/protocol language:
- `PatientController`, `WorkflowEndpoint`
- Verbs: `Get`, `Post`, `Put`, `Delete`

**The rule:** if you see infrastructure vocabulary in the domain layer (like `ISqlPatientStore`
instead of `IPatientRepository`), that is a boundary violation. The domain must not know or
care about SQL, HTTP, or any other technology.

### 3. Interfaces Describe Capabilities, Implementations Describe Strategies

| Interface (what) | Implementation (how) |
|---|---|
| `IMessageQueue` | `PostgresSkipLockedQueue` |
| `IPatientRepository` | `EfCorePatientRepository` |
| `IWorkflowEngine` | `ElsaWorkflowEngine` |
| `IInboundMapper<T>` | `Hl7AdtInboundMapper` |
| `INotificationService` | `SmtpNotificationService` |

The interface name answers: "What can this thing do?"
The implementation name answers: "How does it do it?"

Never name an interface after its (current) implementation. `IPostgresQueue` is wrong even
if Postgres is the only implementation today. Tomorrow it might be Redis. The interface
shouldn't need to change when the strategy changes.

### 4. Methods Match Their Abstraction Level

**Use-case / application service methods** — describe the business action:
```csharp
// Good
Task ProcessInboundMessage(InboundMessage message);
Task OrchestrateWorkflow(WorkflowDefinition definition);
Task<Patient> RegisterNewPatient(PatientRegistration registration);

// Bad — too technical for this layer
Task DeserializeAndInsertMessage(byte[] payload);
Task RunElsaWorkflowWithRetry(string workflowId);
```

**Repository methods** — describe data operations in domain terms:
```csharp
// Good
Task<Patient?> FindById(PatientId id);
Task Save(Patient patient);
Task<IReadOnlyList<Patient>> FindByDepartment(DepartmentId department);

// Bad — leaked implementation
Task<Patient?> ExecuteSqlQueryById(int id);
Task InsertOrUpdateRow(Patient patient);
```

**Infrastructure methods** — free to be technical:
```csharp
// Fine at this level
Task<NpgsqlDataReader> ExecuteQuery(string sql, NpgsqlParameter[] parameters);
Task PostToEndpoint(Uri uri, HttpContent content);
byte[] SerializeToJson<T>(T value);
```

### 5. Specificity Sweet Spot

Be specific enough to disambiguate. No more.

**Too vague:**
- `mapper` — mapper of what?
- `service` — every class is a service if you squint hard enough
- `data` — as opposed to what, non-data?
- `info` — short for "I didn't think about this name"
- `manager` — the word "manager" is where good naming goes to die
- `helper` / `utils` — a class named `Helper` is a code smell wrapped in a suffix

**Too specific:**
- `inboundHl7AdtMessageToCanonicalPatientDomainModelMapper` — a novel, not a name
- `postgresSkipLockedPatientLabResultWorkflowMessageQueue` — just... no

**Just right:**
- `InboundMapper<AdtMessage, Patient>` — the type system carries the specificity
- `PatientRepository` — we know what it persists
- `WorkflowEngine` — we know what it runs
- `MessageQueue` — we know what it queues

### 6. Boolean Names Are Assertions

Booleans should read as true/false assertions in plain language.

```csharp
// Good — reads as a statement
bool isActive;
bool hasBeenProcessed;
bool canRetry;
bool shouldNotify;

// Bad — reads as... what?
bool active;      // "if (active)" vs "if (isActive)" — the second is a sentence
bool processed;   // adjective or verb? ambiguous
bool notification; // that's a noun, not a statement
bool flag;         // flag of what?
```

### 7. Collection Names Are Plural Nouns

```csharp
// Good
IReadOnlyList<Patient> patients;
Dictionary<string, WorkflowStep> stepsByName;
Queue<InboundMessage> pendingMessages;

// Bad
IReadOnlyList<Patient> patientList;  // the type already says "list"
Dictionary<string, WorkflowStep> dict; // never name a thing by its container type
Queue<InboundMessage> queue;          // what kind of queue?
```

### 8. Async Method Names

Suffix with `Async` only on public API boundaries where the caller needs to know. Internal
methods where everything is async anyway — skip the suffix, it's just noise.

```csharp
// Public interface — suffix helps callers
public interface IPatientRepository
{
    Task<Patient?> FindByIdAsync(PatientId id);
    Task SaveAsync(Patient patient);
}

// Internal orchestration — everything is async, suffix adds nothing
private async Task ProcessMessage(InboundMessage message) { ... }
private async Task NotifySubscribers(DomainEvent @event) { ... }
```

### 9. Names That Are Never OK

These names are banned. If you write them, rename immediately:

| Banned | Why | Use instead |
|---|---|---|
| `Manager` | Means nothing. Everything "manages" something. | Name the actual responsibility: `Coordinator`, `Registry`, `Engine`, `Orchestrator` |
| `Helper` / `Utils` | A drawer for code that has no home. | Find the real responsibility and name it. If it's extension methods, use `[Type]Extensions`. |
| `Data` / `Info` | Redundant — all variables hold data. | Name what the data represents: `PatientRecord`, `LabResult`, `DiagnosticReport` |
| `Impl` suffix | `PatientServiceImpl` adds zero information. | Name the strategy: `CachedPatientService`, `DefaultPatientService` |
| `Base` prefix | `BaseRepository` tells you nothing about behavior. | If abstract, the name should describe the shared behavior: `RepositoryWithAuditLog` |
| `My` prefix | `MyService`, `MyClass` — placeholder laziness. | Name it properly from the start. |
| Numbered suffixes | `PatientService2`, `WorkflowV3` — versioned fear. | If it's different, it has a different responsibility. Name that responsibility. |
| Abbreviations | `msg`, `req`, `res`, `ctx`, `proc` — save keystrokes, lose clarity. | `message`, `request`, `response`, `context`, `process`. Your IDE has autocomplete. |
| `Get` on properties | Properties already "get". `GetName` as a property is redundant. | Just `Name`. Reserve `Get` for methods that do meaningful work. |


## Architecture Rules

### Project / Solution Structure

A Clean Architecture .NET solution separates concerns into projects that enforce dependency
direction at compile time. Dependencies flow inward: outer layers depend on inner layers,
never the reverse.

```
Solution/
├── Domain/              # Entities, value objects, domain events, interfaces
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Events/
│   └── Interfaces/      # IPatientRepository, IMessageQueue, etc.
│
├── Application/         # Use cases, orchestration, DTOs, mappers
│   ├── UseCases/        # or Handlers/, Commands/, Queries/
│   ├── Interfaces/      # Application-level ports (IInboundMapper, etc.)
│   ├── DTOs/
│   └── Mappers/
│
├── Infrastructure/      # Implementations of domain + application interfaces
│   ├── Persistence/     # EF Core, Dapper, raw SQL implementations
│   ├── Messaging/       # Queue implementations
│   ├── External/        # HTTP clients, third-party integrations
│   └── Logging/
│
└── Api/                 # Controllers, middleware, DI composition root
    ├── Controllers/
    ├── Middleware/
    └── Configuration/
```

### Dependency Rules (Non-Negotiable)

1. **Domain depends on nothing.** No NuGet packages beyond the BCL. No references to
   Application, Infrastructure, or Api projects. Domain is pure C#.

2. **Application depends only on Domain.** It defines use cases and may define additional
   interfaces (ports). It must not reference Infrastructure or Api.

3. **Infrastructure depends on Domain and Application.** It implements the interfaces defined
   in Domain and Application. This is where NuGet packages for EF Core, Npgsql, Serilog,
   HTTP clients, etc. live.

4. **Api depends on Application and Infrastructure** (for DI registration). It is the
   composition root where everything gets wired together.

5. **Never let a dependency flow outward.** If you need something from an outer layer in an
   inner layer, define an interface in the inner layer and implement it in the outer layer.
   This is Dependency Inversion — the D in SOLID.

### Code Placement Rules

Ask yourself: "If I swapped out the database / message broker / external API, would this
code need to change?"

- **Yes** → Infrastructure.
- **No, but it orchestrates a use case** → Application.
- **No, and it represents a core business concept** → Domain.
- **No, and it handles HTTP concerns** → Api.

### Anti-Patterns to Refuse

**Anemic domain models.** If your entity is just a bag of properties with no behavior, the
domain logic has leaked into the application or infrastructure layer. Entities should
encapsulate business rules.

**God services.** A class named `PatientService` with 40 methods covering registration,
lab results, scheduling, and billing is not a service — it's a monolith. Split by use case.

**Sync-over-async.** Never call `.Result` or `.GetAwaiter().GetResult()` on async code.
This blocks threads and can deadlock. If you see this in a PR, flag it immediately.

**Repository methods that return IQueryable.** This leaks the query implementation to callers
and makes the repository boundary meaningless. Repositories return materialized collections
or single entities.

**Business logic in controllers.** Controllers receive a request, call a use case, and return
a response. That is all. If a controller has an `if` statement about business rules, that
logic belongs in the Application or Domain layer.

**Catching and swallowing exceptions.** `catch (Exception) { }` is never acceptable. Log it,
wrap it, or let it propagate. Silent failures are production nightmares.


## Quick Reference: The Naming Checklist

Before committing any name, run through this:

1. Can I say it out loud in a sentence? ("The InboundMapper converts external messages.")
2. Does it reveal intent, not implementation?
3. Is the vocabulary appropriate for this layer?
4. Is the interface named for the capability, not the strategy?
5. Is it specific enough to disambiguate, but not a novel?
6. Is it free of banned words (Manager, Helper, Utils, Data, Info, Impl)?
7. Do booleans read as assertions? (`isActive`, `hasBeenProcessed`)
8. Are collections plural nouns without type-name suffixes?
9. Would a new team member understand this name without asking?

If any answer is "no" — rename it. Now, not later.

---
> Source: [Kraminius/.NET-C-Clean-Architecture-Skill](https://github.com/Kraminius/.NET-C-Clean-Architecture-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
