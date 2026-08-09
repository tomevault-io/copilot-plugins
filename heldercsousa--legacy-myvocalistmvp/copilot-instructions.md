## legacy-myvocalistmvp

> > **Living Documentation for AI-Assisted Development**

# CLAUDE.md - MyVocaList Project Context

> **Living Documentation for AI-Assisted Development**
> Last Updated: December 23, 2025
> Version: 2.6

---

## 📱 Application Overview

**MyVocaList** is a comprehensive .NET MAUI 8.0 mobile application designed for intelligent karaoke queue management with advanced features and future social network capabilities.

### Core Purpose
Manage karaoke participant queues with intelligent round-based organization, allowing administrators to track participation/absence, reorder singers, and provide real-time queue status information.


## Detailed Objectives

MyVocaList is a .NET MAUI 8.0 application for managing participant queues in karaoke rounds. It allows managing 1 queue at a time, 
enabling the user to register participation/absence of each singer as they reach position 1 in the queue! When all singers who 
entered the queue have participated or been absent, the round is incremented (round 1, round 2, etc). It allows the user to end a 
round even if there are singers in the queue who haven't participated in the round! It also allows enabling the last closed queue 
as a workaround when the user accidentally ends a queue! It allows reverting the round to its last state, in case the user realizes 
there was a registration error or accidentally ended a round! It also allows moving singers to any position in the queue. It enables 
singers to register in the queue autonomously, where the queue administrator receives notifications for each new singer registration! 
It displays the estimated queue completion time, based on the number of singers still pending to sing in the current round. There are 
also 2 queue modes (mechanical karaoke and bandokê - artist/band performs the instrumental). 

The code is being developed by Claude AI, with Helder serving as the architect and auditor of the work, guiding the AI on the 
technical approaches to be adopted and continuously monitoring performance optimization opportunities.

Key technical achievements include: multilingual support (11 languages), robust anti-crash pthread_mutex system, component-based 
architecture with dependency injection, performance-optimized animations with hardware detection, and SQLite database with Entity 
Framework integration.

### Bandokê Queue Mode
Artists/bands can optionally register their song catalog. Song lyrics can be stored in the MyVocaList database or obtained via 
third-party APIs like Genius.com. The app allows the administrator to register or change the song a participant sang at any 
time (as long as the queue is still active). If a singer is going to sing a song not registered in the artist/band catalog, 
the administrator can register the song the singer will perform just before the performance. If there's an internet connection 
and the song lyrics are not available in local data, it will fetch the lyrics via third-party APIs.

### MVP Key Features (English-Only)
- **Queue Management**: One active queue at a time with round-based progression
- **Participation Tracking**: Admin registers singers and marks participation/absence when reaching position 1
- **Round System**: Automatic round increment when all participants complete
- **Flexibility**: End rounds prematurely or revert to last state
- **Admin-Managed Registration**: Each singer registered by admin using standalone device
- **Queue Modes**: Mechanical karaoke and Bandokê (live instrumental)
- **Time Estimation**: Display estimated completion time based on pending singers
- **Multi-language Infrastructure**: 6 languages supported (MVP focuses on English only)

### Future Features (Post-MVP)
- **Singer Autonomy**: Self-registration capability with admin notifications
- **Facial Recognition**: Quick registration for returning singers
- **Song Medleys**: The app will allow bands to register song medleys in the catalog. In this case, lyrics will notbe displayed on screen unless the medley has been previously registered by the band/musician.
- **Song History**: Personalized song suggestions based on past performances
- **Social Network Integration**: Singer profiles, followers, interactions
- **Live Competitions**: Real-time voting, scoring, leaderboards
- **Cloud Synchronization**: Multi-device support with cloud backend

## 🤖 MVP AI Features (AI Lite)

> **Philosophy:** Optional AI enhancements behind feature flags. Disabled by default.
> **Full Documentation:** `Docs/Guides/AI/` folder

### Quick Reference

| Priority | Feature | Time | Status |
|----------|---------|------|--------|
| 1 | Smart Wait Time Estimation | 1-2 wks | Planned |
| 2 | Song Recommendations | 2-3 wks | Planned |
| 3 | Smart Lyrics Search | 2-3 wks | Planned |

### Feature Flags (All disabled by default)
```csharp
// Services/AI/Configuration/AIFeatureFlags.cs
public static class AIFeatureFlags
{
    public static bool EnableAIFeatures => Preferences.Get("ai_features_enabled", false);
    public static bool EnableSmartWaitTime => Preferences.Get("ai_smart_wait_time", false);
    public static bool EnableSongRecommendations => Preferences.Get("ai_song_recommendations", false);
    public static bool EnableLyricsSearch => Preferences.Get("ai_lyrics_search", false);
}
```

### Architecture Overview
- **Python microservices** (FastAPI) hosted on **AWS Lambda** (free tier)
- **C# HTTP clients** in `Services/AI/` folder
- **Graceful fallback** to non-AI when service unavailable

### AI Documentation (Reading Order)
```
Docs/Guides/AI/
├── 1. AI_ENGINEER_ROADMAP.md        ← Start here (master plan)
├── 2. PYTHON_AI_CURRICULUM.md       ← Daily learning curriculum
├── 3. WEEKLY_IMPLEMENTATION_PLAN.md ← Week-by-week tasks
├── 4. AWS_SETUP_GUIDE.md            ← Cloud deployment
├── 5. MAUI_INTEGRATION_GUIDE.md     ← C# integration patterns
└── 6. QUICK_REFERENCE.md            ← Commands cheat sheet
```

### Development Timeline
- **Month 1-2:** Learn Python/ML (while building core MVP pages)
- **Month 3:** Implement Smart Wait Time (first AI feature)
- **Month 4-5:** Song Recommendations + Lyrics Search
- **Month 6:** Polish and enable features gradually

### Future AI (Post-MVP v2.0)
Pitch Detection, Facial Recognition, Voice Commands, Audience Voting
→ Details in AI documentation folder

---

## 👥 Development Team

### **Helder (Project Architect & Technical Auditor)**
- **Role**: Software Architect, Technical Leader, and Quality Auditor
- **Responsibilities**:
  - Defines technical approaches and architectural decisions
  - Guides AI development through strategic technical leadership
  - Conducts code reviews and quality audits
  - Monitors performance optimization opportunities continuously
  - Makes critical decisions on trade-offs (Scoped vs Singleton, architectural patterns)
  - Identifies and prioritizes technical debt and critical issues
  - Manages project complexity and implementation priorities
  - Ensures compliance with .NET MAUI best practices and mobile development standards

### **Claude AI (Code Developer)**
- **Role**: Code Implementation Specialist
- **Responsibilities**:
  - Implements code according to architectural guidelines provided by Helder
  - Develops features, fixes bugs, and creates technical solutions
  - Follows established patterns and coding standards
  - Provides technical analysis and implementation suggestions
  - Creates comprehensive documentation and code comments
  - Performs systematic debugging and troubleshooting
  - Ensures code quality and maintainability

### **Collaborative Process**
The development follows a structured approach where Helder provides strategic direction and technical oversight while Claude AI handles the detailed implementation work. This partnership combines human architectural vision with AI's systematic code development capabilities.

---

## 🏗️ Technical Stack

### Framework & Core Technologies
- **.NET MAUI 8.0**: Cross-platform mobile framework (net8.0-android)
- **C# 13**: Primary programming language (latest features enabled)
- **XAML**: UI markup language
- **SQLite**: Local database storage
- **Entity Framework Core 9.0.6**: ORM for database operations with migrations

### AI/ML Stack (AI Lite - Optional)
- **Python 3.11**: AI service development
- **FastAPI**: REST API framework
- **AWS Lambda**: Serverless hosting (free tier)

### NuGet Packages
```plaintext
- net8.0-android
- net8.0-ios
- net8.0-maccatalyst
- net8.0-windows10.0.19041.0
- Microsoft.EntityFrameworkCore.Sqlite (versão 9.0.6)
- Microsoft.EntityFrameworkCore.Proxies (versão 9.0.6)
- Microsoft.EntityFrameworkCore.Design (versão 9.0.6)
- Microsoft.Maui.Controls (versão 8.0.100)
- Microsoft.Maui.Controls.Xaml (versão 8.0.100)
- Microsoft.Maui.Controls.Capability (versão 8.0.100)
- Microsoft.Extensions.DependencyInjection
- Microsoft.Extensions.Http (for AI service clients)
- Microsoft.Extensions.Http.Polly (for retry policies)
```

### Platform Support
- **Primary Platform**: Android 13+
- **Future Platform**: iOS 16+ (planned)
- **Build System**: .NET CLI / Visual Studio

---

## 🏗️ Architecture Summary

**5-Project Clean Architecture:**
- **Domain** (entities) → **Contracts** (DTOs) → **Services** (business logic) → **Infra.Data** (repositories/EF Core) → **View** (MAUI UI)

**Key Rules:**
- Business logic ONLY in Services layer
- Use ServiceProvider pattern for DI in pages
- Interface + Implementation in SAME folder (no subfolders)
- Repository pattern with EF Core 9.0.6

**Full details:** See `MyVocaList_migration_clean_architecture_guide.md`

---

## 📁 Project Structure
```
MyVocaList.sln
├── MyVocaList.Domain/              # Pure entities
├── MyVocaList.Contracts/           # DTOs
├── MyVocaList.Services/            # Business logic
│   └── AI/                         # AI service clients (when implemented)
│       ├── Abstractions/
│       ├── Implementation/
│       └── Configuration/
├── MyVocaList.Infra.Data/          # EF Core + repositories
└── MyVocaList.View/                # MAUI UI
    ├── Pages (at root, no subfolder!)
    ├── Components/
    ├── Behaviors/
    ├── Resources/Styles/
    └── Platforms/Android/
```

**Critical folders detail:**
- Pages at View ROOT (no Pages/ subfolder)
- Interface + Implementation in same folder
- No /Helpers folder (use Extensions or Components)

--

## 🎯 Current Layer Responsibilities

### **Domain Layer** (`MyVocaList.Domain`)
- **Contains**: Pure business entities (POCOs)
- **Dependencies**: NONE
- **Purpose**: Core business concepts
- **Rules**: 
  - No references to other projects
  - No framework dependencies
  - Only properties and navigation properties
  - **NO business logic** (business logic belongs in Services!)

### **Contracts Layer** (`MyVocaList.Contracts.*`)
- **Contains**: ViewModels, DTOs
- **Dependencies**: NONE
- **Purpose**: Data transfer and presentation abstractions

### **Services Layer** (`MyVocaList.Services.*`)
- **Contains**: ALL business logic and validation
- **Dependencies**: Domain, Contracts, Infrastructure
- **Purpose**: Implement use cases and business rules
- **Responsibilities**:
  - All validation and business rules
  - Coordinate multiple repositories
  - Transaction management
  - Domain ↔ DTO transformations

### **Infrastructure Layer** (`MyVocaList.Infra.*`)
- **Contains**: Data access, utilities
- **Dependencies**: Domain
- **Purpose**: Technical concerns and database access

### **Presentation Layer** (`MyVocaList.View.*`)
- **Contains**: MAUI UI, pages, components
- **Dependencies**: Infrastructure (MauiProgram DI repositories registration), Services (MauiProgram DI services registration and usage), Contracts
- **Purpose**: User interface and interaction

---

## 🔄 When to Modify Each Layer

### Adding New Entity (e.g., "Song")

```
Step 1: Domain Layer
├─ Add Song.cs to MyVocaList.Domain
├─ Define properties and relationships
└─ NO business logic!

Step 2: Infrastructure Layer  
├─ Add SongConfiguration.cs to MyVocaList.Infra.Data.Config
├─ Add ISongRepository.cs & SongRepository.cs (same folder!)
├─ Create migration: dotnet ef migrations add AddSongEntity
└─ Review migration

Step 3: Contracts Layer
├─ Add SongDto.cs to MyVocaList.Contracts.DTOs
└─ Add SongListItemDto.cs to MyVocaList.Contracts.DTOs.List

Step 4: Services Layer
├─ Add ISongService.cs & SongService.cs (same folder!)
├─ Add SongMapper.cs to MyVocaList.Services.Mappers
├─ Implement ALL business logic in service
└─ Register in MauiProgram.cs (DI)

Step 5: Presentation Layer
├─ Add SongPage.xaml/.cs (in View root, no Pages/ subfolder!)
├─ Use ServiceProvider pattern
└─ Bind to DTOs
```

### New Business Rule
**Always Services layer**, never Domain, Infrastructure (Infra.Data.Repositories) or View (directly in pages code-behind) layers!

### New UI Component
```
MyVocaList.View.Components/
├─ Add MyComponent.xaml
├─ Add MyComponent.xaml.cs
└─ Add styles to Resources/Styles/[Component]Styles.xaml
    (Only if styles are VERY specific to this component!)
    (Most styles should be in global Styles.xaml files)
```

---
## 🌍 Localization

**MVP: English-only** (no localization yet)

**Infrastructure supports 6 languages:**
- en (English) - MVP focus
- pt (Portuguese), es (Spanish), fr (French)
- ja (Japanese), ko (Korean)

**NEVER implement localization in MVP** - wait for post-MVP phase.

---

## 💼 Business Logic & Services

### Service Registration (MauiProgram.cs)

```csharp
// === UTILITIES (SINGLETON - stateless) ===
builder.Services.AddSingleton<ITextNormalizer, TextNormalizer>();
builder.Services.AddSingleton<ILanguageService, LanguageService>();

// === REPOSITORIES (SCOPED - database context) ===
builder.Services.AddScoped<IPessoaRepository, PessoaRepository>();
builder.Services.AddScoped<IEstabelecimentoRepository, EstabelecimentoRepository>();
builder.Services.AddScoped<IEventoRepository, EventoRepository>();
builder.Services.AddScoped<IParticipacaoEventoRepository, ParticipacaoEventoRepository>();

// === SERVICES (SCOPED - with state) ===
builder.Services.AddScoped<IPessoaService, PessoaService>();
builder.Services.AddScoped<IEstabelecimentoService, EstabelecimentoService>();
builder.Services.AddScoped<IQueueService, QueueService>();
builder.Services.AddScoped<IDatabaseService, DatabaseService>();

// === PAGES (TRANSIENT - new instance per navigation) ===
builder.Services.AddTransient<SplashPage>();
builder.Services.AddTransient<TonguePage>();
builder.Services.AddTransient<StackPage>();
builder.Services.AddTransient<PersonPage>();
builder.Services.AddTransient<SpotPage>();
builder.Services.AddTransient<SpotFormPage>();
```

---

## 🗂️ Database Architecture

### Entity Framework Core 9.0.6

**Connection String**: 
```csharp
var dbPath = Path.Combine(FileSystem.AppDataDirectory, "myvocalist.db");
options.UseSqlite($"Data Source={dbPath}")
```

### Key Features
- **DatabaseLoadingInterceptor**: Automatic loading indicators + **Auto-trimming string parameters**
- **Text Normalization**: Multilingual search (6 languages)
- **Hybrid Validation**: Input (200 chars) + Database (250 chars)
- **Homonym Handling**: Birthday/Email for disambiguation
- **Case & Accent Insensitive Search**: Database-level collation (NOCASE_NOACCENT)

### Database Best Practices

**⚠️ CRITICAL: Do NOT manually trim strings in EF Core queries!**

The `DatabaseLoadingInterceptor` **automatically trims all string parameters** before query execution. This means:

```csharp
// ❌ WRONG - Manual trimming (redundant and clutters code)
var trimmedName = name.Trim();
return await _context.Estabelecimentos
    .Where(e => e.Nome == trimmedName);

// ✅ CORRECT - Automatic trimming by interceptor
return await _context.Estabelecimentos
    .Where(e => e.Nome == name);  // Interceptor handles trimming!
```

**Why This Matters:**
- ✅ **Cleaner code**: No repetitive `.Trim()` calls
- ✅ **Developer-forget-proof**: Automatic for all queries
- ✅ **Centralized**: Single point of control in interceptor
- ✅ **Performance**: Client-side trimming (no database overhead)

**Collation (Case & Accent Insensitive):**
- **Configured in:** `AppDbContext.OnModelCreating()` via `SetDatabaseCollation()` method
- **Automatic:** Applied to ALL string properties in ALL entities (developer-forget-proof)
- **Current:** SQLite uses custom `NOCASE_NOACCENT` collation (registered in `RegisterCustomCollation()`)
- **Supports:** "João" = "joao" = "JOAO" = "jOãO" (any case/accent combination)
- **Future Migration:** When migrating to SQL Server, simply update `SetDatabaseCollation()`:
  ```csharp
  // SQLite (current)
  property.SetCollation("NOCASE_NOACCENT");

  // SQL Server (future migration)
  property.SetCollation("Latin1_General_CI_AI");  // CI=Case Insensitive, AI=Accent Insensitive
  ```
- **⚠️ IMPORTANT:** All new string properties automatically inherit collation - no manual configuration needed!

---

## ⚡ Performance Best Practices

### **🚫 CRITICAL: Avoid Runtime Reflection**

**Reflection is EXPENSIVE and should NEVER run during user interactions!**

#### **The Problem:**
```csharp
// ❌ BAD - Reflection on EVERY user action (slow!)
public async Task ProcessUserAction(object data)
{
    var type = data.GetType();  // Reflection!
    var properties = type.GetProperties();  // Reflection!

    foreach (var prop in properties)  // Slow loop!
    {
        var value = prop.GetValue(data);  // Reflection!
        await ProcessValue(value);
    }
}
```

**Performance Impact:**
- ❌ **50-100x slower** than direct property access
- ❌ **Garbage collection pressure** (allocations)
- ❌ **Battery drain** on mobile devices
- ❌ **UI lag** during user interactions

#### **The Solution: Pre-compute at Startup**

**✅ GOOD - Reflection ONCE at app startup, cache the results:**
```csharp
// Startup: MauiProgram.cs or App.xaml.cs
public static class TypeCache
{
    private static readonly Dictionary<Type, PropertyInfo[]> _propertyCache = new();

    // ✅ Called ONCE during app initialization
    public static void WarmUpCache()
    {
        var types = new[] { typeof(Pessoa), typeof(Estabelecimento), typeof(Evento) };

        foreach (var type in types)
        {
            _propertyCache[type] = type.GetProperties();  // Reflection once!
        }
    }

    // ✅ Fast lookup (no reflection!)
    public static PropertyInfo[] GetProperties(Type type)
    {
        return _propertyCache.TryGetValue(type, out var props)
            ? props
            : type.GetProperties();  // Fallback (rare)
    }
}

// Runtime: Fast cached access
public async Task ProcessUserAction(object data)
{
    var type = data.GetType();
    var properties = TypeCache.GetProperties(type);  // ✅ Cached! Fast!

    foreach (var prop in properties)
    {
        var value = prop.GetValue(data);
        await ProcessValue(value);
    }
}
```

#### **Alternatives to Reflection:**

**Option 1: Source Generators (Best Performance)**
```csharp
// Zero reflection, compile-time code generation
// Use Roslyn Source Generators for metadata
```

**Option 2: Expression Trees (Good Performance)**
```csharp
// Compiled lambdas - much faster than reflection
var getter = CreateGetter<Pessoa>(p => p.Nome);
```

**Option 3: Pre-generated Mapping Files**
```csharp
// Generate mapping code at build time
// No runtime reflection at all
```

#### **Guidelines:**

| Scenario | Approach | Performance |
|----------|----------|-------------|
| **App startup** | ✅ Reflection OK | One-time cost |
| **User interaction** | ❌ NO reflection | Must be instant |
| **Background task** | ⚠️ Reflection acceptable | Not blocking UI |
| **Hot path (loops)** | ❌ NO reflection | Critical path |

#### **Mobile-Specific Concerns:**

**Why This Matters More on Mobile:**
- 📱 **Limited CPU**: Mobile processors are slower
- 🔋 **Battery life**: Reflection consumes more power
- 💾 **Memory pressure**: GC pressure affects performance
- 📶 **User expectation**: Users expect instant response

#### **Rule of Thumb:**

```
If code runs while user is waiting → NO REFLECTION
If code runs at app startup → REFLECTION OK (cache results!)
If code runs in background → REFLECTION OK (if needed)
```

**⚠️ NEVER use reflection in:**
- UI event handlers (button clicks, text changes)
- Data binding paths (called repeatedly)
- Validation loops (input validation)
- Search/filter operations (user is waiting)
- Animation loops (performance critical)

**✅ Reflection is OK in:**
- App initialization (MauiProgram.cs)
- Dependency injection setup (one-time)
- Migration/seeding (background, one-time)
- Debug/diagnostic tools (not production hot path)

---

## ✅ Validation Strategy

### **Guard Pattern (Repositories/Services)**
```csharp
// ✅ Use Guard for parameter validation
Guard.AgainstNullOrWhiteSpace(nome, nameof(nome));
Guard.AgainstNegativeOrZero(id, nameof(id));
```
**Location:** `Infra/Utils/Guard.cs` | **When:** Method preconditions (fail-fast)

### **FluentValidation (Complex Business Rules)**
```csharp
// ✅ Use for: Homonym validation, duplicate detection, complex DTOs
public class PessoaDtoValidator : AbstractValidator<PessoaDto> { ... }
```
**When:** Cross-field validation, async rules, multiple errors needed | **API:** Essential post-MVP

### **MAUI Behaviors (Simple UI)**
```csharp
// ✅ Use for: Character counters, required fields, basic input validation
```
**When:** UI-level validation, instant feedback

**Rule:** Repository/Service = Guard | Business Logic = FluentValidation | UI = MAUI Behaviors

---

## 🎨 Material Design Quick Reference

**Always use styles (NEVER hardcoded values):**
- Colors: `{StaticResource Primary}`, `{StaticResource OnSurface}`
- Typography: `{StaticResource BodyLarge}`, `{StaticResource TitleMedium}`
- Spacing: Multiples of 8 (16, 24, 32)
- Buttons: MaterialButtonFilled (primary), MaterialButtonOutlined (secondary)

**Full MD3 guidelines:** See `Docs/Guides/MyVocaList_migration_material_design_guide.md`

**Common mistakes to avoid:**
❌ BackgroundColor="#E91E63"  
✅ BackgroundColor="{StaticResource Primary}"

❌ FontSize="16"  
✅ Style="{StaticResource BodyLarge}"

---

## 📋 Code Conventions - CRITICAL

### Language Standard: ENGLISH ONLY

**ABSOLUTE REQUIREMENT - NO EXCEPTIONS:**
- ✅ ALL variable names: English
- ✅ ALL function/method names: English
- ✅ ALL class/interface names: English
- ✅ ALL comments: English
- ✅ ALL UI strings: English
- ✅ ALL database fields: English
- ✅ ALL file names: English

**Current Status:**
- ❌ Legacy code is in Portuguese (being migrated)
- ✅ NEW code must be 100% English
- 🔄 When modifying existing Portuguese code, translate it to English

**Before writing ANY code, ask yourself:**
"Is this in English? If NO, rewrite in English."

**Examples:**

❌ WRONG (Portuguese):
```csharp
public void ValidarUsuario(string nomeUsuario)
{
    // Verifica se usuário existe
    if (string.IsNullOrEmpty(nomeUsuario))
        throw new Exception("Nome de usuário inválido");
}
```

✅ CORRECT (English):
```csharp
public void ValidateUser(string username)
{
    // Check if user exists
    if (string.IsNullOrEmpty(username))
        throw new Exception("Invalid username");
}
```

---

## 🚨 CRITICAL CODING RULES - READ BEFORE WRITING ANY CODE

**⚠️ THESE ARE THE MOST COMMON MISTAKES - NEVER MAKE THEM AGAIN!**

### ❌ ANTIPATTERN #1: Using Console.WriteLine or Debug.WriteLine

**ABSOLUTELY FORBIDDEN - NO EXCEPTIONS:**

```csharp
// ❌ NEVER DO THIS
Console.WriteLine("Loading data...");
Console.WriteLine($"Found {count} items");
Debug.WriteLine($"Error: {ex.Message}");

// ❌ NEVER DO THIS - String interpolation with Serilog
_logger.LogDebug($"Loading {count} items");  // WRONG!

// ✅ ALWAYS DO THIS - Serilog with structured logging
_logger.LogDebug("Loading data");
_logger.LogDebug("Found {Count} items", count);
Logger.Debug("Error occurred: {ErrorMessage}", ex.Message);
```

**Why this matters:**
- Console.WriteLine output is LOST on mobile devices
- Debug.WriteLine only works in debug mode
- Structured logging (Serilog) provides filtering, searching, and analytics
- String interpolation loses structured data benefits

**Rule:** If you type `Console.` or `Debug.` → STOP! Use Serilog instead!

---

### ❌ ANTIPATTERN #2: Unnecessary try-catch Blocks

**DEFAULT RULE: NEVER USE try-catch BLOCKS!**

**GlobalExceptionHandler** catches ALL unhandled exceptions and shows user-friendly messages. Using try-catch blocks **HIDES errors** and makes debugging impossible!

```csharp
// ❌ ANTIPATTERN - Hiding errors from GlobalExceptionHandler
private async Task LoadDataAsync()
{
    try
    {
        var data = await _service.GetDataAsync();
        UpdateUI(data);
    }
    catch (Exception ex)
    {
        Logger.Error(ex, "Error loading data");  // Just logging - not helping!
    }
}

// ✅ CORRECT - Let GlobalExceptionHandler catch it
private async Task LoadDataAsync()
{
    Logger.Debug("Loading data");

    var data = await _service.GetDataAsync();
    UpdateUI(data);

    Logger.Debug("Data loaded successfully");
}
```

**ONLY 4 Acceptable Use Cases for try-catch:**

**1. Expected Operation Cancellation (debouncing)**
```csharp
try
{
    await Task.Delay(300, cancellationToken);
    var results = await _service.SearchAsync(query);
}
catch (OperationCanceledException)
{
    // ✅ Expected - user is still typing
    Logger.Debug("Search cancelled - user still typing");
}
// ❌ DO NOT add catch (Exception ex) here!
```

**2. Hardware-Specific Features (safe to swallow)**
```csharp
try
{
    HapticFeedback.Default.Perform(HapticFeedbackType.Click);
}
catch
{
    // ✅ OK - Hardware not supported, safe to ignore
}
```

**3. User Feedback with Cleanup**
```csharp
await GlobalLoadingOverlay.ShowLoadingAsync("Deleting...");
try
{
    await _service.DeleteAsync(ids);
    await GlobalSnackbar.ShowSuccessAsync("Deleted successfully!");
}
catch (Exception ex)
{
    // ✅ Acceptable - Show user feedback and cleanup UI
    Logger.Error(ex, "Delete failed for IDs: {Ids}", ids);
    await GlobalSnackbar.ShowErrorAsync($"Error: {ex.Message}");
}
finally
{
    await GlobalLoadingOverlay.HideLoadingAsync();  // ✅ Essential cleanup
}
```

**4. Specific Exception with Recovery**
```csharp
try
{
    await _repository.SaveAsync(entity);
}
catch (DbUpdateConcurrencyException ex)
{
    // ✅ Specific exception, specific recovery
    Logger.Warning(ex, "Concurrency conflict - reloading entity {Id}", entity.Id);
    await _repository.ReloadAsync(entity);
    throw;  // Re-throw after recovery attempt
}
// ❌ DO NOT add catch (Exception ex) here!
```

**Quick Decision Rule:**
- If you're writing `catch (Exception ex)` → **STOP! You're doing it wrong!**
- Let GlobalExceptionHandler handle errors (that's why it exists!)
- Only use try-catch for the 4 specific cases above

**Common try-catch Antipatterns to AVOID:**
```csharp
// ❌ ANTIPATTERN #1: Catch-and-log (useless!)
try { await DoSomething(); }
catch (Exception ex) { Logger.Error(ex, "Error"); }

// ❌ ANTIPATTERN #2: Empty catch blocks
try { await DoSomething(); }
catch { }  // Silent failure - NEVER!

// ❌ ANTIPATTERN #3: Catch-and-return-default
try { return await GetData(); }
catch { return new List<Data>(); }  // Hiding errors!

// ❌ ANTIPATTERN #4: Using throw ex (loses stack trace)
catch (Exception ex) { throw ex; }  // ❌ WRONG
catch (Exception ex) { throw; }     // ✅ CORRECT
```

---

### ❌ ANTIPATTERN #3: Manual Parameter Validation

**ALWAYS use Guard pattern for parameter validation - NEVER manual null checks!**

```csharp
// ❌ ANTIPATTERN - Manual validation
public async Task UpdateVenueAsync(int id, string name)
{
    if (id <= 0)
        throw new ArgumentException("ID must be positive");
    if (string.IsNullOrWhiteSpace(name))
        throw new ArgumentException("Name required");

    // ... business logic
}

// ✅ CORRECT - Guard pattern
public async Task UpdateVenueAsync(int id, string name)
{
    Guard.AgainstNegativeOrZero(id, nameof(id));
    Guard.AgainstNullOrWhiteSpace(name, nameof(name));

    // ... business logic
}
```

**Why Guard pattern matters:**
- ✅ Consistent validation across entire codebase
- ✅ Cleaner, more readable code
- ✅ Standard error messages
- ✅ Less boilerplate code
- ✅ Centralized validation logic

**Available Guard methods:**
```csharp
Guard.AgainstNull(value, nameof(value));
Guard.AgainstNullOrWhiteSpace(text, nameof(text));
Guard.AgainstNegativeOrZero(number, nameof(number));
Guard.IsNullOrWhiteSpace(text);  // Returns bool (no exception)
```

**Exception: Validation methods that return tuples:**
```csharp
// ✅ This is OK - validation method returns result, doesn't throw
public (bool isValid, string message) ValidateInput(string input)
{
    if (string.IsNullOrWhiteSpace(input))
        return (false, "Input is required");  // ✅ Returns validation result

    return (true, "");
}
```

---

### 🎯 Before Writing ANY Code - Checklist

**Ask yourself these 3 questions:**

1. **Am I using Console.WriteLine or Debug.WriteLine?**
   - ❌ YES → STOP! Use Serilog instead
   - ✅ NO → Continue

2. **Am I writing a try-catch block?**
   - ❌ YES → Is it one of the 4 acceptable cases? If NO, remove it!
   - ✅ NO → Continue

3. **Am I manually validating parameters (if/throw)?**
   - ❌ YES → STOP! Use Guard pattern instead
   - ✅ NO → Continue

**If you answered ❌ to ANY question → FIX IT BEFORE CONTINUING!**

---

### ServiceProvider Pattern (Critical!)

```csharp
public partial class MyPage : ContentPage
{
    private ServiceProvider? _serviceProvider;
    private IMyService? _myService;

    public MyPage()  // MUST be parameterless!
    {
        InitializeComponent();
    }

    protected override void OnHandlerChanged()
    {
        base.OnHandlerChanged();
        
        if (Handler != null)
        {
            _serviceProvider = ServiceProvider.FromPage(this);
            _myService = _serviceProvider.GetService<IMyService>();
            _ = LoadDataAsync();
        }
    }
}
```

---

## 🔧 Logging & Exception Handling

### Logging (Serilog) - CRITICAL GUIDELINES

#### **Logger Initialization Patterns**

```csharp
// Services (DI - PREFERRED):
public class MyService(ILogger<MyService> logger) { }

// Components/Pages (static - when DI not available):
private static readonly Serilog.ILogger Logger = Log.ForContext<MyComponent>();
```

#### **Structured Logging (MANDATORY)**

**ALWAYS use structured logging templates with placeholders, NEVER string interpolation:**

```csharp
// ✅ CORRECT - Structured logging (allows filtering, searching, analytics)
logger.LogDebug("Loading {Count} items from {Source}", count, source);
logger.LogInformation("User {UserId} created queue {QueueId}", userId, queueId);

// ❌ WRONG - String interpolation (loses structured data)
logger.LogDebug($"Loading {count} items from {source}");
logger.LogInformation($"User {userId} created queue {queueId}");
```

#### **Log Level Guidelines - When to Use Each Level**

**⚠️ CRITICAL RULE: Most logs should be Debug, Information is ONLY for business events!**

---

**🔍 Debug Level** - Routine operations, technical details (90% of your logs)

**Use Debug for:**
- ✅ Constructor execution: `"MyService initialized"`
- ✅ Method entry/exit: `"Starting data load"`, `"Load completed"`
- ✅ Navigation: `"Navigating to {PageName}"`, `"Navigation completed"`
- ✅ Component initialization: `"XAML components initialized"`
- ✅ Service initialization: `"Database service ready"`
- ✅ Database operations: `"Executing query"`, `"Migration applied"`
- ✅ Data loading: `"Loading {Count} items"`
- ✅ UI state changes: `"Search mode activated"`
- ✅ Configuration loading: `"Settings loaded"`
- ✅ Cache operations: `"Cache hit for {Key}"`

**Examples:**
```csharp
// Page lifecycle
Logger.Debug("OnAppearing called");
Logger.Debug("Handler initialized");
Logger.Debug("Services resolved from DI");

// Data operations
_logger.LogDebug("Loading venues with search term: {SearchTerm}", searchTerm);
_logger.LogDebug("Query returned {Count} results", results.Count);
_logger.LogDebug("Database connection verified");

// Navigation
Logger.Debug("Navigating to {TargetPage}", nameof(SpotPage));
Logger.Debug("Navigation to {Page} completed successfully", pageName);
```

---

**📊 Information Level** - Business events, user actions (5-10% of your logs)

**Use Information for:**
- ✅ User actions: `"User {UserId} logged in"`
- ✅ Business operations: `"Queue {QueueId} created by user {UserId}"`
- ✅ Important state changes: `"Round {Round} started with {Count} participants"`
- ✅ Business milestones: `"User {UserId} completed onboarding"`
- ✅ Key business decisions: `"Language {Language} selected by user"`
- ✅ Transaction completion: `"Order {OrderId} processed successfully"`

**Examples:**
```csharp
// User actions (business-relevant)
_logger.LogInformation("User {UserId} selected language: {Language}", userId, language);
_logger.LogInformation("User {UserId} cancelled language selection", userId);

// Business operations
_logger.LogInformation("Queue {QueueId} created at venue {VenueId}", queueId, venueId);
_logger.LogInformation("Singer {SingerId} registered for queue {QueueId}", singerId, queueId);
_logger.LogInformation("Round {Round} completed with {Count} participants", round, count);
```

**⚠️ NEVER use Information for:**
- ❌ Constructor calls: `"MyService initialized"` → Use Debug
- ❌ Page navigation: `"Navigating to SpotPage"` → Use Debug
- ❌ Data loading: `"Loading venues"` → Use Debug
- ❌ Database operations: `"Migration applied"` → Use Debug
- ❌ Service initialization: `"Database ready"` → Use Debug

---

**⚠️ Warning Level** - Recoverable issues, degraded functionality

**Use Warning for:**
- ✅ Fallback scenarios: `"Service unavailable, using cached data"`
- ✅ Deprecated features: `"Using deprecated API, migrate to v2"`
- ✅ Validation failures: `"Invalid input {Input}, using default"`
- ✅ Configuration issues: `"Setting {Key} not found, using default"`
- ✅ Performance issues: `"Query took {Duration}ms, consider optimization"`
- ✅ Missing optional data: `"Profile picture not found for user {UserId}"`

**Examples:**
```csharp
Logger.Warning("Service {ServiceName} not available, using fallback", serviceName);
_logger.LogWarning("Database not initialized after {Attempts} attempts", attempts);
Logger.Warning(ex, "Error checking language via service, falling back to preferences");
```

---

**❌ Error Level** - Exceptions, failures requiring attention

**Use Error for:**
- ✅ Caught exceptions: `"Failed to save venue: {ErrorMessage}"`
- ✅ Operation failures: `"Database migration failed"`
- ✅ External service failures: `"API call to {Service} failed: {Error}"`
- ✅ Data integrity issues: `"Duplicate entry detected for {Key}"`
- ✅ User-impacting errors: `"Failed to load user profile"`

**Examples:**
```csharp
Logger.Error(ex, "Failed to initialize database");
_logger.LogError(ex, "Error saving venue {VenueId}", venueId);
Logger.Error(ex, "Navigation to {Page} failed", pageName);
```

---

**💀 Fatal Level** - Critical failures, application cannot continue

**Use Fatal for:**
- ✅ App initialization failures: `"Critical error during app startup"`
- ✅ Unrecoverable errors: `"Database corrupted, cannot continue"`
- ✅ Total failure scenarios: `"All fallback mechanisms failed"`

**Examples:**
```csharp
Logger.Fatal(ex, "CRITICAL ERROR during application initialization");
Logger.Fatal(ex, "Total failure in navigation fallback");
Logger.Fatal(ex, "Database connection lost and recovery failed");
```

---

#### **Quick Decision Tree**

```
Is this a user action or business event?
  └─ YES → Information
  └─ NO ↓

Is this an exception or error?
  └─ YES → Error (or Fatal if unrecoverable)
  └─ NO ↓

Is this a recoverable issue/fallback?
  └─ YES → Warning
  └─ NO ↓

Is this routine operation/technical detail?
  └─ YES → Debug
```

---

#### **Real-World Examples from Codebase**

```csharp
// ✅ CORRECT Examples:

// App.xaml.cs - Routine operations
Logger.Debug("Configuring runtime environment");
Logger.Debug("XAML components initialized");
Logger.Debug("Essential services initialized");

// SplashPage.xaml.cs - Database operations
Logger.Debug("Starting database initialization");
Logger.Debug("DatabaseService.InitializeDatabaseAsync() completed");
Logger.Debug("Database initialized successfully");

// TonguePage.xaml.cs - Business event vs routine operation
Logger.Information("Language selection cancelled by user");  // Business event!
Logger.Debug("Navigation to StackPage completed");           // Routine operation

// DatabaseService.cs - Database operations
_logger.LogDebug("Starting migration application");
_logger.LogDebug("Migrations applied successfully");
_logger.LogDebug("Database initialized successfully at: {DbPath}", dbPath);
```

---

### Exception Handling - CRITICAL ANTIPATTERN PREVENTION

**⚠️ ABSOLUTE RULES - NO EXCEPTIONS (literally!):**

#### **1. NEVER Use try-catch Blocks (Default Rule)**

**The problem:** Try-catch blocks hide errors from GlobalExceptionHandler, making debugging impossible!

❌ **WRONG - Unnecessary try-catch:**
```csharp
// ❌ BAD - Hiding errors from global handler
private async Task LoadDataAsync()
{
    try
    {
        var data = await _service.GetDataAsync();
        UpdateUI(data);
    }
    catch (Exception ex)
    {
        Logger.Error(ex, "Error loading data");  // Just logging - not helping!
    }
}
```

✅ **CORRECT - Let GlobalExceptionHandler catch it:**
```csharp
// ✅ GOOD - Clean code, errors go to global handler
private async Task LoadDataAsync()
{
    Logger.Debug("Loading data");

    var data = await _service.GetDataAsync();
    UpdateUI(data);

    Logger.Debug("Data loaded successfully");
}
```

**Why this matters:**
- GlobalExceptionHandler shows user-friendly error messages
- Stack traces are preserved for debugging
- Cleaner, more maintainable code
- No silent failures

---

#### **2. ONLY Use try-catch for These Specific Cases**

**✅ Acceptable try-catch scenarios (RARE!):**

**A. Expected Operation Cancellation (debouncing, cancellation tokens):**
```csharp
try
{
    await Task.Delay(300, cancellationToken);
    var results = await _service.SearchAsync(query);
}
catch (OperationCanceledException)
{
    // ✅ Expected - user is still typing
    Logger.Debug("Search cancelled - user still typing");
}
// ❌ DO NOT add catch (Exception ex) here!
```

**B. Hardware-Specific Features (haptics, sensors - acceptable to swallow):**
```csharp
try
{
    HapticFeedback.Default.Perform(HapticFeedbackType.Click);
}
catch
{
    // ✅ OK - Hardware not supported, safe to ignore
}
```

**C. User Feedback with Cleanup (loading overlays, UI feedback):**
```csharp
await GlobalLoadingOverlay.ShowLoadingAsync("Deleting...");
try
{
    await _service.DeleteAsync(ids);
    await GlobalSnackbar.ShowSuccessAsync("Deleted successfully!");
}
catch (Exception ex)
{
    // ✅ Acceptable - Show user feedback and cleanup UI
    Logger.Error(ex, "Delete failed for IDs: {Ids}", ids);
    await GlobalSnackbar.ShowErrorAsync($"Error: {ex.Message}");
}
finally
{
    await GlobalLoadingOverlay.HideLoadingAsync();  // ✅ Essential cleanup
}
```

**D. Specific Exception with Recovery Strategy:**
```csharp
try
{
    await _repository.SaveAsync(entity);
}
catch (DbUpdateConcurrencyException ex)
{
    // ✅ Specific exception, specific recovery
    Logger.Warning(ex, "Concurrency conflict - reloading entity {Id}", entity.Id);
    await _repository.ReloadAsync(entity);
    throw;  // Re-throw after recovery attempt
}
// ❌ DO NOT add catch (Exception ex) here!
```

---

#### **3. NEVER Use Console.WriteLine or Debug.WriteLine - ALWAYS Use Serilog**

**❌ FORBIDDEN:**
```csharp
// ❌ NEVER use Console.WriteLine
Console.WriteLine($"Loading {count} items");

// ❌ NEVER use Debug.WriteLine
System.Diagnostics.Debug.WriteLine($"Error: {ex.Message}");

// ❌ NEVER use string interpolation with Serilog
_logger.LogDebug($"Loading {count} items");  // Loses structured data!
```

**✅ ALWAYS use Serilog with structured logging:**
```csharp
// ✅ Correct - Structured logging
_logger.LogDebug("Loading {Count} items", count);
Logger.Debug("Search completed - found {ResultCount} results", results.Count);

// ✅ Correct - Exception logging with context
_logger.LogError(ex, "Failed to save entity {EntityId}", entity.Id);
```

---

#### **4. ALWAYS Use Guard Pattern for Parameter Validation**

**❌ WRONG - Manual null checks:**
```csharp
public async Task UpdateVenueAsync(int id, string name)
{
    if (id <= 0) throw new ArgumentException("ID must be positive");
    if (string.IsNullOrWhiteSpace(name)) throw new ArgumentException("Name required");

    // ... business logic
}
```

**✅ CORRECT - Guard pattern:**
```csharp
public async Task UpdateVenueAsync(int id, string name)
{
    Guard.AgainstNegativeOrZero(id, nameof(id));
    Guard.AgainstNullOrWhiteSpace(name, nameof(name));

    // ... business logic
}
```

**⚠️ Exception: Validation methods that return tuples:**
```csharp
// ✅ This is OK - validation method returns result, doesn't throw
public (bool isValid, string message) ValidateInput(string input)
{
    if (string.IsNullOrWhiteSpace(input))
        return (false, "Input is required");  // ✅ Returns validation result

    // ... more validation
    return (true, "");
}
```

---

#### **5. Common Antipattern Examples to AVOID**

**❌ ANTIPATTERN #1: Catch-and-log (useless!):**
```csharp
try { await DoSomething(); }
catch (Exception ex) { Logger.Error(ex, "Error"); }  // ❌ Just logging? Let global handler do it!
```

**❌ ANTIPATTERN #2: Empty catch blocks:**
```csharp
try { await DoSomething(); }
catch { }  // ❌ Silent failure - NEVER acceptable (except hardware features)
```

**❌ ANTIPATTERN #3: Catch-and-return-default:**
```csharp
try { return await GetData(); }
catch { return new List<Data>(); }  // ❌ Hiding errors!
```

**❌ ANTIPATTERN #4: Using throw ex (loses stack trace):**
```csharp
catch (Exception ex) { throw ex; }  // ❌ Loses original stack trace
```

**✅ CORRECT - Use throw; to preserve stack trace:**
```csharp
catch (SpecificException ex)
{
    Logger.Error(ex, "Context");
    throw;  // ✅ Preserves stack trace
}
```

---

#### **Quick Reference - When Can I Use try-catch?**

| Scenario | Use try-catch? | Why? |
|----------|---------------|------|
| **General business logic** | ❌ NO | Let GlobalExceptionHandler handle it |
| **Database operations** | ❌ NO | Let GlobalExceptionHandler handle it |
| **Service calls** | ❌ NO | Let GlobalExceptionHandler handle it |
| **Navigation** | ❌ NO | Let GlobalExceptionHandler handle it |
| **UI updates** | ❌ NO | Let GlobalExceptionHandler handle it |
| **OperationCanceledException** | ✅ YES | Expected for debouncing |
| **Haptic/sensor features** | ✅ YES | Hardware-specific, safe to ignore |
| **With loading overlay cleanup** | ✅ YES | Need to hide overlay + show feedback |
| **Specific exception with recovery** | ✅ YES | DbUpdateConcurrency, network retry, etc. |

**Rule of thumb:** If you're writing `catch (Exception ex)` → **STOP! You're doing it wrong!**

---

## 🔄 Behaviors for Code Reuse

**Avoid code duplication using Behaviors:**

**SmartPageLifecycleBehavior** - Page lifecycle + loading + navbar


```xml
<behaviors:SmartPageLifecycleBehavior 
    NavBar="{x:Reference CrudNavBar}"
    LoadDataCommand="{Binding LoadDataCommand}"
    UseGlobalLoading="True" />
```

**SafeNavigationBehavior** - Thread-safe navigation with debounce
```xml
<behaviors:SafeNavigationBehavior 
    TargetPageType="{x:Type local:SpotFormPage}"
    DebounceMilliseconds="800" />
```

**NavBarBehavior** - Dynamic button generation (used internally by components)

**When to use:** Repeated code in 3+ pages, configurable via XAML
**When NOT to use:** Page-specific logic, business rules

**Detailed documentation:** Full examples and troubleshooting in project knowledge base.

---

## 🔄 Git Workflow & Commit Guidelines

**CRITICAL: Always commit after successful build/testing!**

### When to Commit

**✅ MUST commit when:**
1. **Build succeeds** - After running `dotnet build` with no errors
2. **Tests pass** - After running tests successfully (when applicable)
3. **Feature is complete** - After finishing a logical unit of work
4. **Before switching tasks** - Before starting a different feature/fix

**❌ DO NOT commit when:**
- Build has errors
- Tests are failing
- Code is half-finished or broken
- Local settings files are modified (`.claude/settings.local.json`)

### Commit Process

**Standard workflow:**
```bash
# 1. Verify build succeeds
dotnet build

# 2. Check what files changed
git status

# 3. Stage only relevant files (exclude local settings)
git add <file1> <file2> <file3>

# 4. Commit with descriptive message
git commit -m "$(cat <<'EOF'
<type>: <short summary>

- <detailed change 1>
- <detailed change 2>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

**Commit message format:**
- **Type**: `fix:`, `feat:`, `refactor:`, `docs:`, `perf:`, `test:`
- **Short summary**: What changed (50 chars max)
- **Details**: Bullet points explaining the changes
- **Always include**: Claude Code attribution footer

### Files to Exclude from Commits

**NEVER commit these files:**
- `.claude/settings.local.json` - Local Claude Code settings
- `bin/`, `obj/` - Build output (already in .gitignore)
- `.vs/` - Visual Studio settings (already in .gitignore)
- User-specific IDE settings

---
## 📋 Documentation Standards

**Changelog Location:** `Docs/Changelog/changelog.md`

**CRITICAL Rules:**
- ✅ **ALWAYS update changelog.md after completing ANY task** (no exceptions!)
- ✅ **ALWAYS commit changes after successful build** (no exceptions!)
- ✅ **Create guide files in `Docs/Guides/`** and add to solution (like changelog.md)
- ✅ **Format:** `- **MM/dd/yyyy** - Type - Description` (Type: Enhancement or Fix)

---

**Last Updated**: December 23, 2025
**Version**: 2.6
**Maintained by**: Helder (Architect) + Claude AI (Developer)

---
> Source: [heldercsousa/Legacy_MyVocaListMVP](https://github.com/heldercsousa/Legacy_MyVocaListMVP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
