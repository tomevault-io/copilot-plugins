## budgetlens

> > Versão: 1.1.0 | Última atualização: 2026-04-06

# CLAUDE.md — BudgetLens
> Versão: 1.1.0 | Última atualização: 2026-04-06

---

## 🏗️ Visão Geral do Projeto

**BudgetLens** é um analisador de extrato bancário com IA. O usuário faz upload de um PDF ou CSV do seu extrato, o sistema categoriza as transações automaticamente via LLM, identifica padrões de comportamento financeiro e gera insights que aplicativos bancários tradicionais não oferecem.

### Objetivo de negócio
- Parsing robusto de extratos de diferentes bancos (cada banco tem formato próprio)
- Categorização automática via LLM com cache inteligente (evitar chamadas redundantes)
- Sistema de aprendizado: quando o usuário corrige uma categoria, o sistema aprende e aplica em transações futuras similares
- Geração de insights contextuais em PT-BR

### Stack
| Camada | Tecnologia | Versão |
|---|---|---|
| Linguagem | Java | 17 |
| Framework principal | Spring Boot | 3.3.x |
| Pipeline de dados | Spring Batch | 5.x |
| Persistência | Spring Data JPA + Hibernate | 6.x |
| Banco de dados | PostgreSQL | 16 |
| Migrations | Flyway | 10.x |
| Segurança | Spring Security + JWT (jjwt) | 0.12.x |
| HTTP Client (LLM) | RestClient (Spring 6 nativo) | — |
| Build | Maven | 3.9.x |
| Containerização | Docker Compose | — |
| Frontend | Angular + Angular Material | 17 |
| Testes | JUnit 5 + Mockito + Testcontainers | — |

### Arquitetura
Clean Architecture com 4 camadas:
- `domain` — modelos de domínio e interfaces de repositório (sem dependências externas)
- `application` — serviços de aplicação e DTOs
- `infrastructure` — implementações: JPA, batch, parsers, AI client
- `presentation` — controllers REST

---

## 📁 Estrutura de Pastas

```
budgetlens/
├── backend/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/budgetlens/
│   │   │   │   ├── domain/
│   │   │   │   │   ├── model/              # Entidades de domínio puras (sem anotações JPA)
│   │   │   │   │   │   ├── Transaction.java
│   │   │   │   │   │   ├── Category.java
│   │   │   │   │   │   ├── Statement.java
│   │   │   │   │   │   ├── UserCorrection.java
│   │   │   │   │   │   └── Insight.java
│   │   │   │   │   └── port/               # Interfaces de repositório (contrato)
│   │   │   │   │       ├── TransactionRepository.java
│   │   │   │   │       ├── CategoryRepository.java
│   │   │   │   │       ├── StatementRepository.java
│   │   │   │   │       └── UserCorrectionRepository.java
│   │   │   │   ├── application/
│   │   │   │   │   ├── service/            # Lógica de negócio
│   │   │   │   │   │   ├── StatementParserService.java
│   │   │   │   │   │   ├── CategorizationService.java
│   │   │   │   │   │   ├── InsightGeneratorService.java
│   │   │   │   │   │   ├── UserCorrectionService.java
│   │   │   │   │   │   └── StatementSummaryService.java
│   │   │   │   │   └── dto/                # Request/Response DTOs (fronteira da API)
│   │   │   │   ├── infrastructure/
│   │   │   │   │   ├── batch/              # Spring Batch: Jobs, Steps, Readers, Writers
│   │   │   │   │   │   ├── CategorizationJob.java
│   │   │   │   │   │   ├── CategorizationProcessor.java
│   │   │   │   │   │   └── InsightJobListener.java
│   │   │   │   │   ├── ai/                 # Integração com LLM
│   │   │   │   │   │   ├── OpenAIClient.java
│   │   │   │   │   │   ├── PromptBuilder.java
│   │   │   │   │   │   └── CategorizationCache.java
│   │   │   │   │   ├── parser/             # Parsers de extrato (padrão Strategy)
│   │   │   │   │   │   ├── StatementParser.java       (interface)
│   │   │   │   │   │   ├── NubankCsvParser.java
│   │   │   │   │   │   ├── GenericBankCsvParser.java
│   │   │   │   │   │   └── ParserFactory.java
│   │   │   │   │   └── persistence/        # Implementações JPA + entidades
│   │   │   │   │       ├── entity/         # Entidades JPA (anotações aqui, não no domínio)
│   │   │   │   │       ├── TransactionCategoryJpaRepository.java  # findAllByStatementId, deleteAllByStatementId
│   │   │   │   │       ├── TransactionJpaRepository.java          # findAllByStatementId(Pageable), filtro por categoria
│   │   │   │   │       └── UserCorrectionJpaRepository.java       # LIKE pattern%, ORDER BY comprimento desc
│   │   │   │   └── presentation/
│   │   │   │       └── controller/
│   │   │   │           ├── AuthController.java          # POST /api/auth/register, /login
│   │   │   │           ├── StatementController.java     # GET /api/statements, /{id}/summary, /insights, /transactions
│   │   │   │           ├── TransactionController.java   # PATCH /api/transactions/{id}/category
│   │   │   │           └── CategoryController.java      # GET /api/categories
│   │   │   └── resources/
│   │   │       ├── application.yml
│   │   │       ├── application-dev.yml
│   │   │       └── db/migration/           # Scripts Flyway (V1__..., V2__..., etc.)
│   │   └── test/
│   │       ├── java/com/budgetlens/        # Espelha estrutura de main/
│   │       └── resources/
│   │           └── extratos/               # Arquivos CSV/PDF de teste
│   └── pom.xml
├── frontend/
│   └── src/app/
│       ├── features/
│       │   ├── upload/
│       │   ├── dashboard/
│       │   └── insights/
│       └── shared/
├── docker-compose.yml
└── CLAUDE.md
```

---

## ⚙️ Configurações do Ambiente

### Pré-requisitos
- Java 17 (verificar: `java -version`)
- Maven 3.9+ (verificar: `mvn -version`)
- Docker Desktop rodando
- Node.js 18+ para o frontend Angular

### Subir o ambiente de desenvolvimento
```bash
# 1. Subir banco de dados
docker-compose up -d

# 2. Verificar se o banco subiu
docker-compose ps

# 3. Rodar a API
cd backend
mvn spring-boot:run -Dspring.profiles.active=dev

# 4. Rodar o frontend (em outro terminal)
cd frontend
npm install
ng serve
```

### Variáveis de ambiente necessárias
```bash
# Obrigatório — chave da OpenAI
export OPENAI_API_KEY=sk-...

# Opcional — sobrescrever configurações do banco
export DB_URL=jdbc:postgresql://localhost:5432/budgetlens
export DB_USERNAME=budgetlens
export DB_PASSWORD=budgetlens123
```

### docker-compose.yml
```yaml
services:
  postgres:
    image: postgres:16
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: budgetlens
      POSTGRES_USER: budgetlens
      POSTGRES_PASSWORD: budgetlens123
    volumes:
      - postgres_data:/var/lib/postgresql/data
volumes:
  postgres_data:
```

### application-dev.yml
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/budgetlens
    username: budgetlens
    password: budgetlens123
  batch:
    job:
      enabled: false  # NUNCA rodar jobs no startup — sempre via endpoint

openai:
  api:
    key: ${OPENAI_API_KEY}
  model: gpt-4o-mini
```

---

## 📐 Padrões do Projeto

### Nomenclatura
- Classes: `PascalCase` em inglês
- Métodos e variáveis: `camelCase` em inglês
- Constantes: `UPPER_SNAKE_CASE`
- Tabelas e colunas SQL: `snake_case`
- DTOs: sufixo `Request` ou `Response` (ex: `StatementUploadResponse`)
- Entidades JPA: sufixo `Entity` (ex: `TransactionEntity`) — distingue do modelo de domínio
- Serviços: sufixo `Service` (ex: `CategorizationService`)
- Repositórios JPA: sufixo `JpaRepository` (ex: `TransactionJpaRepository`)

### Injeção de dependência
**Sempre via construtor — nunca `@Autowired` em campo.**
```java
// CORRETO
@Service
public class CategorizationService {
    private final TransactionRepository transactionRepository;
    private final OpenAIClient openAIClient;

    public CategorizationService(TransactionRepository transactionRepository,
                                  OpenAIClient openAIClient) {
        this.transactionRepository = transactionRepository;
        this.openAIClient = openAIClient;
    }
}

// ERRADO — nunca fazer isso
@Autowired
private TransactionRepository transactionRepository;
```

### DTOs na fronteira
Toda comunicação entre Controller e Service usa DTOs. Modelos de domínio nunca saem da camada de aplicação.
```
Controller → recebe RequestDTO → chama Service → recebe ResponseDTO → retorna ao cliente
Service → usa modelos de domínio internamente → nunca expõe entidades JPA para fora
```

### Padrão Strategy para parsers
Cada banco tem seu parser concreto. A interface é `StatementParser`.
- `supports(MultipartFile file)` — retorna true se consegue parsear o arquivo
- `parse(MultipartFile file)` — retorna lista de `TransactionDTO`
- `ParserFactory` itera pelos parsers registrados como `@Component` e delega ao primeiro que suporta

### Chain of Responsibility para categorização
A busca de categoria segue esta prioridade (nunca inverter):
1. **UserCorrection** — correção explícita do usuário (source = `USER_CORRECTION`)
2. **LearningCache** — padrão aprendido de correções anteriores (source = `LEARNED`)
3. **CategorizationCache** — resultado anterior da LLM cacheado (source = `AI_CACHED`)
4. **OpenAI API** — chamada fresca à LLM (source = `AI`)
5. **Fallback** — categoria "Outros" se tudo falhar

### Normalização de descrição para cache/aprendizado
Antes de usar uma descrição como chave de cache ou padrão de correção, normalizar:
```java
// Normalização padrão:
// 1. lowercase
// 2. remover números (exceto se forem parte do nome — ex: "7-Eleven" vira "7eleven")
// 3. remover sufixos alfanuméricos após * (ex: "IFOOD*ABC123" vira "ifood")
// 4. trim e colapsar espaços múltiplos
String normalize(String description) {
    return description.toLowerCase()
        .replaceAll("\\*[a-z0-9]+", "")   // Remove sufixo após asterisco
        .replaceAll("\\s+", " ")
        .trim();
}
```

### Testes
- Toda lógica de negócio **deve ter testes** — nenhum serviço sem cobertura
- Nomenclatura: `[Classe]Test` para unit, `[Classe]IT` para integration
- Testcontainers para testes que precisam do banco real
- Mocks via Mockito para dependências externas (OpenAI, etc.)
- Arquivos CSV de fixture em `src/test/resources/extratos/`

---

## 🏛️ ADRs — Decisões de Arquitetura

### ADR-001: Spring Batch para o pipeline de categorização
**Contexto**: Categorizar centenas de transações de um extrato de uma vez exige processamento robusto, com retry, skip de erros e rastreabilidade.
**Decisão**: Usar Spring Batch com Job/Step/Reader/Processor/Writer em vez de um loop simples no Service.
**Consequências**: + Retry automático, + rastreabilidade via batch metadata, + pausar/retomar jobs | - Maior complexidade inicial, - curva de aprendizado do Batch
**Status**: Aceita

### ADR-002: Cache em memória no MVP (não Redis)
**Contexto**: Redis adiciona complexidade de infra. No MVP, o cache só precisa sobreviver enquanto o job está rodando.
**Decisão**: `ConcurrentHashMap` como cache de embeddings/categorizações para o MVP. Migrar para Redis quando houver necessidade de cache persistente entre reinicializações.
**Consequências**: + Simplicidade, + zero infra adicional | - Cache perdido ao reiniciar a aplicação
**Status**: Aceita (revisar após MVP)

### ADR-003: Entidades JPA separadas dos modelos de domínio
**Contexto**: Annotations JPA poluem os modelos de domínio e criam acoplamento com a infraestrutura.
**Decisão**: Duas classes separadas — `Transaction` (domínio puro) e `TransactionEntity` (JPA na camada de infraestrutura). Mappers fazem a conversão.
**Consequências**: + Domínio limpo e testável, + liberdade de mudar o banco sem afetar o domínio | - Mais classes, - mappers adicionais
**Status**: Aceita

### ADR-004: gpt-4o-mini como modelo LLM
**Contexto**: Categorizar uma transação é uma tarefa simples. Modelos mais potentes seriam desperdício de custo.
**Decisão**: `gpt-4o-mini` com `temperature: 0` e `max_tokens: 20`. Resposta esperada: apenas o nome da categoria.
**Consequências**: + Custo baixíssimo (~$0.0001 por 100 transações), + respostas rápidas e determinísticas | - Capacidade de raciocínio limitada (suficiente para categorização)
**Status**: Aceita

### ADR-005: `spring.batch.job.enabled=false` em todos os perfis
**Contexto**: Spring Batch, por padrão, roda todos os jobs cadastrados no startup da aplicação.
**Decisão**: Sempre desabilitar auto-start. Jobs só rodam via `JobLauncher` acionado pelo endpoint `POST /api/statements/{id}/process`.
**Consequências**: + Controle total sobre quando os jobs rodam | - Requer endpoint explícito para disparar
**Status**: Aceita — **nunca reverter essa configuração**

### ADR-006: Geração de insights como efeito colateral do batch (não etapa separada)
**Contexto**: Insights precisam ser gerados logo após a categorização terminar, sem exigir chamada extra do cliente.
**Decisão**: `StatementStatusListener` (JobExecutionListener) detecta quando o job termina com sucesso e chama `InsightGeneratorService.generateInsights()` automaticamente. Falha no insight não reverte a categorização — o statement fica `CATEGORIZED` mesmo que os insights falhem.
**Consequências**: + UX simplificada (cliente só faz polling de status), + categorização não é comprometida por falha de insight | - Falha silenciosa nos insights requer monitoramento de log
**Status**: Aceita

### ADR-007: `findMatchingByUserIdAndDescription` com ORDER BY comprimento desc
**Contexto**: Um usuário pode ter padrões de correção sobrepostos — "ifood" e "ifood delivery" — e o padrão mais específico deve ter prioridade.
**Decisão**: Query JPQL com `LIKE pattern%` retorna todos os padrões que fazem match, ordenados por `LENGTH(pattern) DESC`. O primeiro resultado é o mais específico.
**Consequências**: + Comportamento intuitivo, + sem lógica extra no Java | - Query mais custosa (aceitável — user_corrections é uma tabela pequena)
**Status**: Aceita

---

## 🐛 Erros Conhecidos e Como Evitá-los

### [2026-03-27] Armadilha: Spring Batch rodando no startup
**O que pode acontecer**: Se `spring.batch.job.enabled` não for explicitamente `false`, o Batch tenta rodar jobs ao iniciar a aplicação — antes do banco estar populado com dados.
**Como prevenir**: Sempre verificar `application.yml` e `application-dev.yml` — essa config deve existir em todos os perfis.
```yaml
spring:
  batch:
    job:
      enabled: false
```

### [2026-03-27] Armadilha: N+1 em relacionamentos JPA
**O que pode acontecer**: Carregar uma lista de `Statement` e depois acessar `statement.getTransactions()` dentro de um loop causa uma query por statement.
**Como prevenir**: Repositórios que carregam coleções aninhadas devem usar `JOIN FETCH` ou `@EntityGraph`.
**Convenção adotada**: Métodos de repositório que carregam relacionamentos têm sufixo `WithDetails`:
```java
@Query("SELECT s FROM StatementEntity s JOIN FETCH s.transactions WHERE s.id = :id")
Optional<StatementEntity> findByIdWithDetails(@Param("id") UUID id);
```

### [2026-03-27] Armadilha: API key da OpenAI em logs
**O que pode acontecer**: Logar o objeto de configuração ou a requisição HTTP completa pode expor a API key.
**Como prevenir**:
- Nunca fazer `log.info("Config: {}", openAIConfig)`
- Nunca logar headers de requisições HTTP para a OpenAI
- Usar `${OPENAI_API_KEY}` no yml — nunca hardcoded

### [2026-04-06] Armadilha: Insight falho não pode derrubar a categorização
**O que pode acontecer**: Se `InsightGeneratorService` lançar exceção dentro do `JobExecutionListener`, o job pode ser marcado como falho mesmo com todas as transações categorizadas.
**Como prevenir**: O bloco de geração de insights no listener **deve ter try/catch** — logar o erro, marcar o statement como `CATEGORIZED` (não `INSIGHTS_READY`) e seguir em frente. O usuário vê as categorias; os insights aparecem em retry posterior.
```java
try {
    insightGeneratorService.generateInsights(statementId);
} catch (Exception e) {
    log.error("Insight generation failed for statement {}: {}", statementId, e.getMessage());
    // statement permanece CATEGORIZED — não relançar
}
```

### [2026-04-06] Armadilha: LLM retornando JSON com texto extra nos insights
**O que pode acontecer**: O modelo pode retornar ```json [...] ``` com backticks ou prefixo de texto antes do array JSON.
**Como prevenir**: `InsightGeneratorService` deve tentar extrair o array JSON do conteúdo bruto antes de parsear. Implementar fallback linha a linha se o JSON falhar:
```java
// 1. Tentar parsear direto
// 2. Extrair via regex o primeiro [...] encontrado
// 3. Fallback: dividir por \n e usar cada linha como insight individual
```

### [2026-04-06] Armadilha: `deleteAllByStatementId` sem JOIN explícito
**O que pode acontecer**: `TransactionCategoryEntity` não tem `statementId` diretamente — está em `TransactionEntity`. Um delete simples por statementId não funciona sem JOIN.
**Como prevenir**: Usar JPQL com subquery:
```java
@Modifying
@Query("DELETE FROM TransactionCategoryEntity tc WHERE tc.transaction.id IN " +
       "(SELECT t.id FROM TransactionEntity t WHERE t.statement.id = :statementId)")
void deleteAllByStatementId(@Param("statementId") UUID statementId);
```
**O que pode acontecer**: `Double.parseDouble("1.234,56")` lança `NumberFormatException`. Extratos brasileiros usam vírgula como decimal.
**Como prevenir**: Usar método utilitário para parsing de moeda:
```java
BigDecimal parseAmount(String raw) {
    return new BigDecimal(raw.trim()
        .replace("R$", "")
        .replace(".", "")   // separador de milhar
        .replace(",", ".")  // decimal
        .trim());
}
```
**Sempre usar `BigDecimal` para valores financeiros — nunca `double` ou `float`.**

---

## 🚀 Otimizações e Performance

### Cache de categorização por descrição normalizada
Antes de chamar a OpenAI, normalizar a descrição e verificar o cache. Transações com descrições similares (mesmo estabelecimento, pedido diferente) reutilizam a categorização anterior.
- Exemplo: "IFOOD*PEDIDO123" e "IFOOD*PEDIDO456" → mesma chave normalizada "ifood" → uma chamada só

### Paginação no Spring Batch Reader
`JpaPagingItemReader` configurado com `pageSize: 50`. Nunca carregar todas as transações de um extrato em memória de uma vez — extratos grandes podem ter 500+ transações.

### Batch load de TransactionCategory via `findAllByStatementId`
Em vez de carregar `TransactionCategory` por transação individual (N+1), usar `findAllByStatementId` que retorna todas de uma vez e montar um Map em memória para lookup O(1).
- Implementado em `TransactionCategoryJpaRepository` — usar este padrão em qualquer consulta de resumo/summary.

---

## 🤖 Agentes: Casos de Uso Confirmados

| Agente | Tarefa executada com sucesso | Data |
|---|---|---|
| `engineering-data-engineer` + `engineering-senior-developer` | Parser Strategy (NubankCsvParser, GenericBankCsvParser, ParserFactory) | 2026-04-06 |
| `engineering-data-engineer` + `engineering-senior-developer` | Spring Batch pipeline (CategorizationJob, Reader, Processor, Writer) | 2026-04-06 |
| `engineering-ai-engineer` + `engineering-senior-developer` | OpenAIClient, CategorizationCache, InsightGeneratorService | 2026-04-06 |
| `engineering-backend-architect` + `engineering-senior-developer` | UserCorrectionService, CategoryService, StatementQueryService | 2026-04-06 |
| `engineering-security-engineer` + `engineering-backend-architect` | SecurityConfig + JwtAuthenticationFilter, AuthController | 2026-04-06 |
| `testing-api-tester` + `testing-test-results-analyzer` | UserCorrectionServiceTest, InsightGeneratorServiceTest, StatementSummaryServiceTest, AuthControllerTest | 2026-04-06 |
| `engineering-database-optimizer` + `engineering-senior-developer` | findAllByStatementId (batch load), deleteAllByStatementId (JPQL join), findMatchingByUserIdAndDescription (LIKE + ORDER BY comprimento) | 2026-04-06 |

---

## 📚 Regras de Negócio Relevantes

### Categorias do sistema
As seguintes categorias são pré-criadas e não podem ser deletadas pelo usuário:
`Alimentação`, `Transporte`, `Saúde`, `Lazer`, `Educação`, `Moradia`, `Salário`, `Outros`

### Prioridade de categorização
Correção do usuário > Padrão aprendido > Cache da LLM > LLM ao vivo > Fallback "Outros"
Nunca inverter essa prioridade.

### Normalização de padrões de aprendizado
O padrão salvo em `user_corrections.transaction_description_pattern` é a descrição normalizada — não a descrição bruta. Isso permite que "IFOOD*PEDIDO999" e "IFOOD*ABC" sejam reconhecidos pelo mesmo padrão "ifood".

### Ownership — segurança de dados
Todo acesso a dados (statements, transações, correções) deve verificar que o recurso pertence ao `userId` extraído do JWT. Nunca confiar em IDs vindos do body da requisição para determinar ownership.

### Limite de linhas por extrato
Máximo de 500 transações por arquivo. Retornar `422 Unprocessable Entity` com mensagem clara se ultrapassar.

### Padrão mais específico tem prioridade no aprendizado
`UserCorrectionJpaRepository` retorna padrões ordenados por `LENGTH(pattern) DESC`. O primeiro resultado é sempre o padrão mais específico — nunca alterar essa ordenação.

### `generateText` no OpenAIClient (método adicional)
Além de `categorize()`, o `OpenAIClient` expõe `generateText(String prompt): String` para uso livre pelo `InsightGeneratorService`. Esse método usa `max_tokens: 500` e `temperature: 0.3` (mais criativo que a categorização).

### Falha de insight não bloqueia o fluxo
Se `InsightGeneratorService` falhar, o statement permanece com status `CATEGORIZED` (não `INSIGHTS_READY`). O dashboard mostra categorias normalmente — insights aparecem como ausentes. Nunca propagar a exceção para fora do listener.

### Status do Statement
Ciclo de vida: `PENDING` → `PROCESSING` → `CATEGORIZED` → `INSIGHTS_READY`
- `PENDING`: upload feito, aguardando processamento
- `PROCESSING`: job de categorização rodando
- `CATEGORIZED`: todas as transações categorizadas, insights ainda não gerados
- `INSIGHTS_READY`: pronto para exibir no dashboard

---

## 🔗 Dependências e Integrações Relevantes

### OpenAI API
- **Endpoint**: `https://api.openai.com/v1/chat/completions`
- **Modelo**: `gpt-4o-mini`
- **Autenticação**: Header `Authorization: Bearer ${OPENAI_API_KEY}`
- **Rate limit**: 500 RPM no tier gratuito — implementar retry com backoff de 1s (máx 3 tentativas)
- **`categorize()`**: `max_tokens: 20`, `temperature: 0` — resposta deve ser APENAS o nome da categoria
- **`generateText()`**: `max_tokens: 500`, `temperature: 0.3` — usado pelo InsightGeneratorService para insights em PT-BR

### Flyway
- Scripts em `src/main/resources/db/migration/`
- Nomenclatura obrigatória: `V{número}__{descrição_com_underscores}.sql`
  - Exemplo: `V1__create_users_table.sql`
- Nunca editar um script já aplicado — criar novo script de alteração

### Spring Batch metadata tables
O Spring Batch cria suas próprias tabelas no banco (`BATCH_JOB_INSTANCE`, `BATCH_JOB_EXECUTION`, etc.). Não tentar criar essas tabelas manualmente — o Batch gerencia automaticamente.

---

## 📝 Changelog do CLAUDE.md

| Versão | Data | O que mudou |
|---|---|---|
| 1.0.0 | 2026-03-27 | Criação inicial — estrutura completa do projeto BudgetLens |
| 1.1.0 | 2026-04-06 | MVP concluído: estrutura de pastas atualizada com arquivos reais, ADR-006 e ADR-007, 3 novas armadilhas documentadas, agentes confirmados preenchidos, otimização de batch load, regras de negócio do InsightGeneratorService e OpenAIClient |

---
> Source: [joaogabriel43/budgetlens](https://github.com/joaogabriel43/budgetlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
