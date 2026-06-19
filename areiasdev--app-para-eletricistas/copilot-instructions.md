## app-para-eletricistas

> Estamos a construir uma **web app SaaS** para eletricistas, técnicos de AVAC e pequenas empresas de manutenção em Portugal. O produto resolve problemas reais e concretos:

# ⚡ TécnicoApp — Skill / Especificação de Projeto para Claude Code

## Visão Geral

Estamos a construir uma **web app SaaS** para eletricistas, técnicos de AVAC e pequenas empresas de manutenção em Portugal. O produto resolve problemas reais e concretos:

- Perda de tempo a criar orçamentos à mão ou no Excel
- Falta de histórico de clientes e equipamentos
- Esquecimento de manutenções periódicas
- Falta de profissionalismo no contacto com o cliente (sem assinatura digital, sem PDF, etc.)

O target é o técnico individual ou empresa com 1-10 funcionários que paga €20-60/mês.

---

## Stack Técnica

| Camada | Tecnologia |
|---|---|
| **Frontend** | Next.js 14+ (App Router) + Tailwind CSS + shadcn/ui |
| **Backend / API** | ASP.NET Core 8 Web API (C#) |
| **Base de dados** | PostgreSQL |
| **ORM** | Entity Framework Core 8 |
| **Autenticação** | ASP.NET Core Identity + JWT Bearer |
| **Armazenamento de ficheiros** | Azure Blob Storage ou AWS S3 |
| **Geração de PDFs** | QuestPDF (C#) |
| **Email transacional** | Resend (via HTTP) ou SendGrid |
| **Jobs / Agendamento** | Hangfire (alertas de manutenção, emails) |
| **Pagamentos** | Stripe .NET SDK |
| **Deploy Backend** | Azure App Service ou Railway |
| **Deploy Frontend** | Vercel |
| **Containerização** | Docker + docker-compose para dev local |

> Para app mobile futura: React Native com Expo consumindo a mesma API .NET.

---

## Arquitetura do Backend (.NET)

### Estrutura de Pastas

```
TecnicoApp.sln
├── src/
│   ├── TecnicoApp.API/              # Projeto Web API (entry point)
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   ├── Program.cs
│   │   └── appsettings.json
│   │
│   ├── TecnicoApp.Application/      # Lógica de negócio (Use Cases)
│   │   ├── Features/
│   │   │   ├── Quotes/
│   │   │   │   ├── Commands/        # CreateQuoteCommand, UpdateQuoteCommand...
│   │   │   │   ├── Queries/         # GetQuoteByIdQuery, GetQuoteListQuery...
│   │   │   │   └── Handlers/
│   │   │   ├── Clients/
│   │   │   ├── Equipment/
│   │   │   └── Interventions/
│   │   ├── Common/
│   │   │   ├── Interfaces/          # IRepository, ICurrentUser, IEmailService...
│   │   │   ├── Behaviors/           # ValidationBehavior, LoggingBehavior (MediatR)
│   │   │   └── Exceptions/          # NotFoundException, ValidationException...
│   │   └── DTOs/
│   │
│   ├── TecnicoApp.Domain/           # Entidades, Value Objects, Enums (sem dependências)
│   │   ├── Entities/
│   │   ├── Enums/
│   │   ├── Events/                  # Domain Events
│   │   └── ValueObjects/
│   │
│   └── TecnicoApp.Infrastructure/   # Implementações concretas
│       ├── Persistence/
│       │   ├── AppDbContext.cs
│       │   ├── Configurations/      # IEntityTypeConfiguration<T>
│       │   └── Migrations/
│       ├── Services/                # EmailService, PdfService, StorageService...
│       └── Identity/
│
└── tests/
    ├── TecnicoApp.UnitTests/
    ├── TecnicoApp.IntegrationTests/
    └── TecnicoApp.FunctionalTests/
```

Esta estrutura segue **Clean Architecture** com **CQRS via MediatR**.

### Packages NuGet Principais

```xml
<!-- TecnicoApp.API -->
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.*" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="6.*" />
<PackageReference Include="Serilog.AspNetCore" Version="8.*" />

<!-- TecnicoApp.Application -->
<PackageReference Include="MediatR" Version="12.*" />
<PackageReference Include="FluentValidation.DependencyInjectionExtensions" Version="11.*" />
<PackageReference Include="AutoMapper" Version="13.*" />
<PackageReference Include="Ardalis.Result" Version="9.*" />

<!-- TecnicoApp.Infrastructure -->
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.*" />
<PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="8.*" />
<PackageReference Include="Hangfire.AspNetCore" Version="1.*" />
<PackageReference Include="QuestPDF" Version="2024.*" />
<PackageReference Include="Stripe.net" Version="45.*" />
<PackageReference Include="Azure.Storage.Blobs" Version="12.*" />
```

---

## Entidades do Domínio (C#)

```csharp
// Domain/Entities/BaseEntity.cs
public abstract class BaseEntity
{
    public Guid Id { get; init; } = Guid.NewGuid();
    public DateTime CreatedAt { get; init; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
}

// Domain/Entities/Client.cs
public class Client : BaseEntity
{
    public string Name { get; set; } = string.Empty;
    public string? Nif { get; set; }
    public string? Email { get; set; }
    public string? Phone { get; set; }
    public Address? Address { get; set; }         // Value Object
    public string? Notes { get; set; }
    public Guid UserId { get; set; }

    public ICollection<Equipment> Equipment { get; set; } = [];
    public ICollection<Quote> Quotes { get; set; } = [];
    public ICollection<Intervention> Interventions { get; set; } = [];
}

// Domain/ValueObjects/Address.cs
public record Address(
    string Street,
    string City,
    string PostalCode,
    string Country = "Portugal"
);

// Domain/Entities/Quote.cs
public class Quote : BaseEntity
{
    public string Number { get; set; } = string.Empty;   // ORC-2025-0042
    public QuoteStatus Status { get; set; } = QuoteStatus.Draft;
    public decimal? Discount { get; set; }
    public string? Notes { get; set; }
    public DateTime? ValidUntil { get; set; }
    public DateTime? SignedAt { get; set; }
    public string? SignatureUrl { get; set; }
    public string? PdfUrl { get; set; }

    public Guid ClientId { get; set; }
    public Client Client { get; set; } = null!;
    public Guid UserId { get; set; }

    public ICollection<QuoteLine> Lines { get; set; } = [];

    // Propriedades calculadas (não mapeadas para DB)
    public decimal SubTotal => Lines.Sum(l => l.Quantity * l.UnitPrice);
    public decimal VatTotal => Lines.Sum(l => l.Quantity * l.UnitPrice * (l.VatRate / 100));
    public decimal Total => SubTotal + VatTotal - (Discount ?? 0);
}

// Domain/Entities/Equipment.cs
public class Equipment : BaseEntity
{
    public string Type { get; set; } = string.Empty;
    public string? Brand { get; set; }
    public string? Model { get; set; }
    public string? SerialNumber { get; set; }
    public DateTime? InstalledAt { get; set; }
    public DateTime? NextMaintenance { get; set; }
    public string? Notes { get; set; }
    public List<string> Photos { get; set; } = [];

    public Guid ClientId { get; set; }
    public Client Client { get; set; } = null!;
    public ICollection<Intervention> Interventions { get; set; } = [];
}

// Domain/Enums/
public enum QuoteStatus { Draft, Sent, Accepted, Rejected, Invoiced }
public enum InterventionStatus { Scheduled, InProgress, Completed }
public enum Plan { Free, Pro, Team }
```

---

## Boas Práticas de Código

---

### 🏗️ Arquitetura & Design

**Clean Architecture — regra de dependências:**
- `Domain` não depende de nada
- `Application` depende apenas de `Domain`
- `Infrastructure` e `API` dependem de `Application`
- Nunca referenciar `Infrastructure` diretamente de `API` (apenas via DI)

**CQRS com MediatR — regra de ouro:**
- Toda a lógica de negócio vive em Commands e Queries — NUNCA nos Controllers
- Controllers são thin: recebem request, chamam `_mediator.Send(command)`, devolvem resultado
- Um Handler por Command/Query, sem exceções

```csharp
// ✅ CORRETO — Controller thin
[HttpPost]
public async Task<IActionResult> Create(CreateQuoteRequest request, CancellationToken ct)
{
    var command = _mapper.Map<CreateQuoteCommand>(request);
    var result = await _mediator.Send(command, ct);
    return result.IsSuccess ? Ok(result.Value) : result.ToActionResult();
}

// ❌ ERRADO — lógica no controller
[HttpPost]
public async Task<IActionResult> Create(CreateQuoteRequest request)
{
    var quote = new Quote { ... };
    _db.Quotes.Add(quote);
    await _db.SaveChangesAsync();
    // enviar email aqui, calcular número, etc.
    return Ok(quote);
}
```

**Princípios gerais:**
- **SRP** — cada classe tem uma única razão para mudar
- **OCP** — aberto para extensão, fechado para modificação (usar interfaces e injeção)
- **DIP** — depender de abstrações, nunca de implementações concretas
- **DRY** — extrair lógica repetida para métodos/serviços partilhados
- **YAGNI** — não construir o que não é necessário agora; o MVP primeiro

---

### 🧹 C# — Regras Gerais

**Nomenclatura:**
- Classes, Métodos, Propriedades: `PascalCase`
- Variáveis locais e parâmetros: `camelCase`
- Campos privados: `_camelCase`
- Constantes: `PascalCase` (convenção moderna C#)
- Interfaces: prefixo `I` (ex: `IQuoteRepository`)
- Métodos assíncronos: sufixo `Async` (ex: `GetQuoteByIdAsync`)

**Tipos e Nulabilidade:**
- Ativar `<Nullable>enable</Nullable>` em todos os projetos
- Usar `record` para DTOs e Value Objects (imutabilidade garantida)
- Usar `sealed` em classes que não devem ser herdadas
- Usar `required` em propriedades obrigatórias (C# 11+)
- Preferir `IReadOnlyList<T>` em retornos públicos de coleções
- Nunca retornar `null` de métodos de negócio — usar `Result<T>` (Ardalis.Result)

```csharp
// ✅ CORRETO
public record CreateQuoteDto(
    Guid ClientId,
    List<QuoteLineDto> Lines,
    string? Notes = null
);

// ✅ CORRETO — sem null, com Result
public async Task<Result<QuoteDto>> Handle(GetQuoteByIdQuery query, CancellationToken ct)
{
    var quote = await _repo.GetByIdAsync(query.Id, ct);
    if (quote is null)
        return Result.NotFound($"Orçamento {query.Id} não encontrado.");

    return Result.Success(_mapper.Map<QuoteDto>(quote));
}
```

**Async/Await — regras críticas:**
- Toda operação de I/O (DB, HTTP, ficheiros) deve ser `async`
- Sempre passar e usar `CancellationToken` em handlers e repositórios
- Nunca usar `.Result` ou `.Wait()` — causa deadlocks em contextos ASP.NET
- Usar `ConfigureAwait(false)` em libraries; não é necessário em ASP.NET Core apps

```csharp
// ✅ CORRETO
public async Task<List<Client>> GetAllAsync(CancellationToken ct)
    => await _db.Clients.AsNoTracking().ToListAsync(ct);

// ❌ ERRADO — bloqueia a thread, potencial deadlock
public List<Client> GetAll()
    => _db.Clients.ToListAsync().Result;
```

**LINQ e EF Core:**
- Usar `AsNoTracking()` em TODAS as queries de leitura
- Nunca fazer `Include()` desnecessários — carregar só o que é preciso para aquele caso
- Usar `Select()` para projetar diretamente em DTOs (evita hidratação de entidades completas)
- Paginação sempre com `Skip()` + `Take()` — nunca trazer tudo para memória e filtrar depois

```csharp
// ✅ CORRETO — projeção, paginação, sem tracking
var quotes = await _db.Quotes
    .AsNoTracking()
    .Where(q => q.UserId == _currentUser.UserId)
    .OrderByDescending(q => q.CreatedAt)
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .Select(q => new QuoteListItemDto(q.Id, q.Number, q.Status, q.Total, q.CreatedAt))
    .ToListAsync(ct);

// ❌ ERRADO — traz tudo, filtra em memória
var all = await _db.Quotes.ToListAsync();
var filtered = all.Where(q => q.UserId == userId).ToList();
```

**Tratamento de Erros:**
- Usar `Result<T>` (Ardalis.Result) para erros de negócio esperados — nunca lançar exceção para fluxo normal
- Exceções apenas para situações verdadeiramente inesperadas (bug, infra down, etc.)
- Global exception middleware apanha tudo o que não foi tratado e devolve `ProblemDetails`
- Sempre logar com Serilog incluindo contexto estruturado (userId, quoteId, etc.)

```csharp
// ✅ CORRETO — erro de negócio com Result, sem exceção
if (quote.Status != QuoteStatus.Draft)
    return Result.Conflict("Só é possível editar orçamentos em rascunho.");

// ✅ CORRETO — log estruturado
_logger.LogWarning("Tentativa de editar orçamento {QuoteId} com estado {Status} por utilizador {UserId}",
    quote.Id, quote.Status, _currentUser.UserId);

// ❌ ERRADO — try/catch genérico que engole erros
try { ... }
catch (Exception) { return Result.Error("Algo correu mal."); }
```

**Validação com FluentValidation:**
- Toda entrada de dados validada com um Validator dedicado
- Validator registado via DI e executado automaticamente no `ValidationBehavior` (MediatR pipeline)
- Nunca validar manualmente dentro dos Handlers

```csharp
public class CreateQuoteCommandValidator : AbstractValidator<CreateQuoteCommand>
{
    public CreateQuoteCommandValidator()
    {
        RuleFor(x => x.ClientId).NotEmpty();
        RuleFor(x => x.Lines)
            .NotEmpty().WithMessage("O orçamento deve ter pelo menos uma linha.");
        RuleForEach(x => x.Lines).ChildRules(line =>
        {
            line.RuleFor(l => l.Description).NotEmpty().MaximumLength(500);
            line.RuleFor(l => l.Quantity).GreaterThan(0);
            line.RuleFor(l => l.UnitPrice).GreaterThanOrEqualTo(0);
            line.RuleFor(l => l.VatRate).InclusiveBetween(0, 100);
        });
    }
}
```

**Segurança — regras inegociáveis:**
- Nunca confiar no `UserId` vindo do body do request — extrair SEMPRE do JWT via `ICurrentUserService`
- Verificar ownership antes de qualquer operação: o recurso pertence ao utilizador autenticado?
- Secrets em `appsettings.json` apenas para dev local; em produção usar Azure Key Vault ou env vars
- Nunca logar passwords, tokens ou dados pessoais sensíveis

```csharp
// ✅ CORRETO — UserId do JWT, verificação de ownership
public async Task<Result<QuoteDto>> Handle(GetQuoteByIdQuery query, CancellationToken ct)
{
    var quote = await _repo.GetByIdAsync(query.QuoteId, ct);
    if (quote is null || quote.UserId != _currentUser.UserId)
        return Result.NotFound(); // não revelar se existe mas não é deles

    return Result.Success(_mapper.Map<QuoteDto>(quote));
}

// ❌ ERRADO — UserId do body
public async Task<Result> Handle(DeleteQuoteCommand command, CancellationToken ct)
{
    var quote = await _repo.GetByIdAsync(command.QuoteId, ct);
    if (quote.UserId != command.UserIdFromBody)  // NUNCA fazer isto
        return Result.Forbidden();
}
```

---

### 🗃️ Entity Framework Core

**Configurações:**
- Usar `IEntityTypeConfiguration<T>` para cada entidade — nunca Data Annotations nas classes de domínio
- Migrations nomeadas descritivamente: `AddEquipmentPhotosColumn`, não `Migration20250412`
- Índices para colunas frequentemente filtradas ou ordenadas

```csharp
public class QuoteConfiguration : IEntityTypeConfiguration<Quote>
{
    public void Configure(EntityTypeBuilder<Quote> builder)
    {
        builder.HasKey(q => q.Id);
        builder.Property(q => q.Number).IsRequired().HasMaxLength(20);
        builder.Property(q => q.Status).HasConversion<string>();
        builder.Property(q => q.Discount).HasColumnType("decimal(10,2)");
        builder.HasMany(q => q.Lines)
               .WithOne(l => l.Quote)
               .OnDelete(DeleteBehavior.Cascade);
        builder.HasIndex(q => new { q.UserId, q.CreatedAt }); // query frequente
        builder.Ignore(q => q.SubTotal);  // propriedade calculada, não persistida
        builder.Ignore(q => q.VatTotal);
        builder.Ignore(q => q.Total);
    }
}
```

**Padrão Repository:**
```csharp
public interface IRepository<T> where T : BaseEntity
{
    Task<T?> GetByIdAsync(Guid id, CancellationToken ct = default);
    Task<List<T>> GetAllAsync(CancellationToken ct = default);
    Task AddAsync(T entity, CancellationToken ct = default);
    void Update(T entity);
    void Delete(T entity);
    Task SaveChangesAsync(CancellationToken ct = default);
}
```

---

### 🌐 API — Contratos & Convenções

**Endpoints RESTful:**
```
GET    /api/v1/quotes              → lista paginada
GET    /api/v1/quotes/{id}         → detalhe
POST   /api/v1/quotes              → criar (201 + Location header)
PUT    /api/v1/quotes/{id}         → atualizar completo
PATCH  /api/v1/quotes/{id}/status  → atualizar apenas estado
DELETE /api/v1/quotes/{id}         → apagar

POST   /api/v1/quotes/{id}/send    → enviar para cliente por email
POST   /api/v1/quotes/{id}/sign    → endpoint público para assinar
GET    /api/v1/quotes/{id}/pdf     → descarregar PDF
```

**Respostas de erro (RFC 7807 — ProblemDetails):**
```json
{
  "type": "https://tecnicoapp.pt/errors/not-found",
  "title": "Orçamento não encontrado",
  "status": 404,
  "detail": "Não existe orçamento com o ID fornecido.",
  "traceId": "00-abc123..."
}
```

**Sempre:**
- Versionar a API: `/api/v1/`
- Usar `[ProducesResponseType]` nos controllers (Swagger legível)
- Retornar `201 Created` com `Location` header ao criar recursos
- Nunca expor IDs internos de base de dados em mensagens de erro para o utilizador final

---

### ⚛️ Frontend (Next.js / TypeScript)

**Estrutura de pastas:**
```
src/
├── app/
│   ├── (auth)/                 # login, registo (sem sidebar)
│   ├── (dashboard)/            # área autenticada com layout
│   │   ├── quotes/
│   │   ├── clients/
│   │   └── equipment/
│   └── sign/[token]/           # página pública de assinatura
├── components/
│   ├── ui/                     # shadcn/ui — não modificar
│   ├── shared/                 # Button, Modal, DataTable, PageHeader...
│   └── features/               # QuoteForm, ClientCard, EquipmentList...
├── lib/
│   ├── api/                    # funções de fetch tipadas para a API .NET
│   ├── utils/                  # formatCurrency, formatDate, validateNif
│   └── validations/            # schemas Zod
├── hooks/                      # useQuotes, useClients, useCurrentUser...
├── stores/                     # Zustand (estado global leve, ex: UI state)
└── types/                      # TypeScript types e interfaces partilhados
```

**Regras TypeScript:**
- `strict: true` no `tsconfig.json` — sem excepções
- Nunca usar `any` — usar `unknown` se o tipo for incerto, depois narrowing com type guards
- Preferir `type` para unions/intersections, `interface` para shape de objectos/contratos
- Exportar tipos junto com os componentes/funções que os usam

```typescript
// ✅ CORRETO
type QuoteStatus = 'Draft' | 'Sent' | 'Accepted' | 'Rejected' | 'Invoiced'

interface Quote {
  id: string
  number: string
  status: QuoteStatus
  total: number
  client: Pick<Client, 'id' | 'name'>
  createdAt: string
}

// ❌ ERRADO
const processQuote = (quote: any) => { ... }
const updateStatus = (id: any, status: any) => { ... }
```

**Fetch & Estado do Servidor (React Query):**
- Usar TanStack Query para TODAS as chamadas à API — nunca `useEffect` para fetch
- `queryKey` sempre incluir parâmetros relevantes
- Mutations com `invalidateQueries` em `onSuccess` para manter cache sincronizado

```typescript
// ✅ CORRETO
const { data: quotes, isLoading } = useQuery({
  queryKey: ['quotes', { page, status }],
  queryFn: () => api.quotes.list({ page, status }),
})

const createQuote = useMutation({
  mutationFn: api.quotes.create,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['quotes'] })
    toast.success('Orçamento criado com sucesso!')
  },
  onError: (err) => {
    toast.error(getErrorMessage(err))
  },
})
```

**Componentes:**
- Pequenos e com uma única responsabilidade
- Props explícitas com tipos definidos (nunca spread de props não tipado)
- Evitar prop drilling além de 2 níveis — usar React Context ou Zustand

```typescript
// ✅ CORRETO
interface QuoteStatusBadgeProps {
  status: QuoteStatus
  className?: string
}

function QuoteStatusBadge({ status, className }: QuoteStatusBadgeProps) {
  const config = STATUS_CONFIG[status]
  return (
    <Badge className={cn(config.className, className)}>
      {config.label}
    </Badge>
  )
}
```

**Formatação para Portugal:**
```typescript
// lib/utils/formatters.ts

export const formatCurrency = (value: number): string =>
  new Intl.NumberFormat('pt-PT', { style: 'currency', currency: 'EUR' }).format(value)
// → "1.234,50 €"

export const formatDate = (date: string | Date): string =>
  new Intl.DateTimeFormat('pt-PT').format(new Date(date))
// → "12/04/2025"

export const validateNif = (nif: string): boolean => {
  if (!/^\d{9}$/.test(nif)) return false
  const digits = nif.split('').map(Number)
  const checksum = digits.slice(0, 8).reduce((sum, d, i) => sum + d * (9 - i), 0)
  return (11 - (checksum % 11)) % 10 === digits[8]
}
```

---

### 🧪 Testes

**Backend (xUnit + NSubstitute + FluentAssertions):**
- Unit tests: Handlers, Validators, lógica de domínio
- Integration tests: repositórios e endpoints com `WebApplicationFactory`
- Nomear testes: `MethodName_Scenario_ExpectedResult`

```csharp
public class CreateQuoteHandlerTests
{
    [Fact]
    public async Task Handle_WithValidCommand_ReturnsCreatedQuote()
    {
        // Arrange
        var repo = Substitute.For<IQuoteRepository>();
        var handler = new CreateQuoteHandler(repo, ...);
        var command = new CreateQuoteCommand { ClientId = Guid.NewGuid(), Lines = [...] };

        // Act
        var result = await handler.Handle(command, CancellationToken.None);

        // Assert
        result.IsSuccess.Should().BeTrue();
        result.Value.Should().NotBeNull();
        await repo.Received(1).AddAsync(Arg.Any<Quote>(), Arg.Any<CancellationToken>());
    }
}
```

**Frontend (Vitest + Testing Library):**
- Testar comportamento, não implementação
- Usar `msw` para mockar chamadas à API nos testes de componente
- Testar formulários com interações reais (fill, click, assert)

---

### 📋 Git & Qualidade

**Conventional Commits:**
```
feat(quotes): adicionar assinatura digital
fix(pdf): corrigir cálculo de IVA a 6%
chore(deps): atualizar QuestPDF para 2024.3
refactor(clients): extrair lógica para ClientRepository
test(quotes): adicionar testes ao CreateQuoteHandler
docs: atualizar SKILL.md com stack .NET
```

**Antes de cada commit:**
- `dotnet build` sem warnings (tratar warnings como errors em CI: `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`)
- `dotnet test` a verde
- `tsc --noEmit` sem erros no frontend
- `eslint` sem erros no frontend

**CI mínimo (GitHub Actions):**
```yaml
jobs:
  backend:
    steps:
      - run: dotnet restore
      - run: dotnet build --no-restore -warnaserror
      - run: dotnet test --no-build --verbosity normal
  frontend:
    steps:
      - run: npm ci
      - run: npm run type-check
      - run: npm run lint
      - run: npm run build
```

---

## Módulos do Produto

### 1. 👤 Autenticação & Onboarding
- Registo com email + password; JWT Bearer com refresh token
- Onboarding de 3 passos: nome da empresa, NIF, tipo de serviço
- Upload de logótipo (Blob Storage)

### 2. 🧑‍🤝‍🧑 Gestão de Clientes
- CRUD com pesquisa e filtros
- Vista de detalhe com histórico completo (orçamentos, intervenções, equipamentos)

### 3. 📋 Orçamentos *(módulo core)*
**Fluxo:** Selecionar cliente → Adicionar linhas (descrição, qtd, preço, IVA) → Preview PDF → Enviar email

**Estados:** `Rascunho` → `Enviado` → `Aceite` / `Recusado` → `Faturado`

**PDF inclui:** logótipo, dados empresa + NIF, dados cliente + NIF, número (ORC-2025-0042), tabela com subtotais e IVA, total, condições, campo de assinatura

**Assinatura digital (Pro):** link único e seguro → cliente assina no ecrã → PDF final com assinatura embebida → estado muda para `Aceite`

### 4. 🔧 Equipamentos
- Associado a cliente: tipo, marca, modelo, nº série, data instalação, próxima manutenção, fotos, notas
- Job Hangfire: alerta automático 7 dias antes da manutenção

### 5. 🗓️ Intervenções / Ordens de Serviço
- Associar a cliente + equipamento(s) + orçamento (opcional)
- Fotos antes/depois, materiais, descrição técnica
- Estados: `Agendada` → `Em curso` → `Concluída`
- Gerar PDF de relatório de intervenção (QuestPDF)

### 6. 🔔 Notificações (Hangfire)
- Email ao técnico 7 dias antes de manutenção
- Email ao cliente (toggle por cliente)
- Notificação quando cliente assina orçamento
- Resumo semanal de orçamentos pendentes e intervenções

### 7. 📊 Dashboard
- Total orçamentos este mês (€ e qtd), taxa de aceitação
- Intervenções da semana, próximas manutenções
- Gráfico de orçamentos por estado

### 8. 💳 Planos & Pagamentos (Stripe)

| Plano | Preço | Limites |
|---|---|---|
| **Grátis** | €0/mês | 5 orçamentos/mês, 10 clientes, sem assinatura digital |
| **Pro** | €29/mês | Ilimitado + assinatura digital + alertas automáticos |
| **Equipa** | €59/mês | Pro + até 5 técnicos + marca branca nos PDFs |

---

## Prioridade de Desenvolvimento (MVP)

1. **Setup** — solução .NET, Next.js, PostgreSQL com Docker
2. **Auth** — registo, login, JWT, refresh token
3. **Clientes** — CRUD completo
4. **Orçamentos** — criação, listagem, estados (sem assinatura ainda)
5. **PDF de orçamento** — QuestPDF, envio por email
6. **Dashboard básico**
7. **Equipamentos + Alertas Hangfire**
8. **Intervenções / Ordens de Serviço**
9. **Assinatura digital** (feature Pro)
10. **Stripe + Planos**

---

## Comandos para Iniciar o Projeto

### Backend (.NET)
```bash
dotnet new sln -n TecnicoApp
dotnet new webapi -n TecnicoApp.API -o src/TecnicoApp.API
dotnet new classlib -n TecnicoApp.Application -o src/TecnicoApp.Application
dotnet new classlib -n TecnicoApp.Domain -o src/TecnicoApp.Domain
dotnet new classlib -n TecnicoApp.Infrastructure -o src/TecnicoApp.Infrastructure
dotnet sln add src/**/*.csproj
```

### Frontend
```bash
npx create-next-app@latest tecnico-app --typescript --tailwind --eslint --app --src-dir
cd tecnico-app
npx shadcn@latest init
npm install @tanstack/react-query axios zod react-hook-form @hookform/resolvers zustand
```

### Docker (dev local)
```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: tecnicoapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

---

## Tom e Linguagem da UI

- **Português Europeu** em toda a UI (não brasileiro)
- Datas: `DD/MM/AAAA` via locale `pt-PT`
- Moeda: `1.234,50 €` via `Intl.NumberFormat('pt-PT')`
- Mensagens de erro humanas: *"Não foi possível guardar o orçamento. Tenta novamente."*
- Botões com verbos claros: "Criar Orçamento", "Enviar para Cliente", "Marcar como Aceite"
- Nunca expor stack traces, IDs de base de dados ou mensagens técnicas na UI

---
> Source: [areiasdev/App-para-Eletricistas](https://github.com/areiasdev/App-para-Eletricistas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
