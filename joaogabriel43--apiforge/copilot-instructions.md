## apiforge

> * **Concluído**: Maio de 2026

# CLAUDE.md - APIForge Memória Persistente

* **Version**: 1.0.0
* **Concluído**: Maio de 2026
* **Status**: Concluído & Consolidado

Este documento serve como memória persistente e guia definitivo de desenvolvimento para o projeto **APIForge**. Ele estabelece as regras arquiteturais, decisões técnicas (ADRs), padrões de projeto, definições da stack e o resumo executivo para fins de portfólio.

---

## 🚀 Visão Geral do Projeto
O **APIForge** é um gerador inteligente de APIs REST a partir de schemas SQL. O sistema lê, analisa e interpreta definições de tabelas SQL (DDLs) e gera automaticamente estruturas de APIs completas, modulares, padronizadas e prontas para execução sob o framework Spring Boot.

---

## 🏛️ Estrutura de Diretórios (Clean Architecture)

A estrutura de pacotes principal reside em `src/main/java/com/apiforge/` e reflete estritamente as camadas da **Clean Architecture**:

```
com.apiforge/
│
├── ApiForgeApplication.java    # Classe de bootstrapping do Spring Boot (Root)
│
├── domain/                     # Camada de Domínio (Pureza de Negócios)
│   ├── model/                  # Entidades de negócio e Objetos de Valor
│   ├── exception/              # Exceções de regras de negócio
│   └── repository/             # Interfaces (Ports) de persistência e serviços externos
│
├── application/                # Camada de Aplicação (Casos de Uso)
│   ├── usecase/                # Implementação dos fluxos de casos de uso
│   ├── dto/                    # Modelos de transferência interna (Commands/Queries)
│   └── exception/              # Exceções específicas da aplicação
│
├── infrastructure/             # Camada de Infraestrutura (Detalhes Técnicos)
│   ├── config/                 # Beans de configuração (Spring, P6Spy, etc.)
│   ├── db/                     # Entidades JPA, Repositórios Spring Data e Flyway Migrations
│   └── adapter/                # Implementações concretas de serviços externos / persistência
│
└── presentation/               # Camada de Apresentação (Interface Externa)
    ├── controller/             # Controllers REST do Spring MVC exposing endpoints
    ├── dto/                    # DTOs de Request e Response da API
    ├── mapper/                 # Conversores entre DTOs de Apresentação e DTOs de Casos de Uso
    └── exception/              # Handler global de exceções e mapeador de HTTP Status
```

### Estrutura do Frontend (Angular Playground)

O ecossistema client reside na pasta `frontend/src/` e segue uma arquitetura modular orientada a componentes reativos:

```
frontend/src/
│
├── main.ts                     # Ponto de entrada bootstrap do Angular standalone
├── styles.css                  # Folha de estilos central contendo diretivas Tailwind CSS
│
└── app/
    ├── app.config.ts           # Configurações globais (Monaco, HTTP Client, Roteador)
    ├── app.component.ts        # Controlador reativo principal (form validation, SSE events, memory leak preventions)
    ├── app.component.html      # Layout grid com split File Tree e Code Viewer
    │
    └── core/                   # Núcleo da aplicação
        ├── models/             # Tipagens e interfaces TypeScript (GenerationOptions, SSE events)
        └── services/           # Serviços de comunicação HTTP/SSE reativos
```

---

## 📋 Padrões do Projeto

1. **Clean Architecture Estrita**:
   * O **Domínio** é o centro e não tem conhecimento de frameworks, bancos de dados ou qualquer detalhe técnico externo (sem anotações Spring ou JPA).
   * A **Aplicação** gerencia os casos de uso e depende puramente do domínio e de interfaces (Ports).
   * A **Infraestrutura** implementa os adaptadores concretos e lida com frameworks diretamente.
   * A **Apresentação** expõe endpoints REST e converte os payloads para chamadas de aplicação.
2. **TDD (Test-Driven Development)**:
   * Escreva os testes unitários e de propriedades antes ou em paralelo às regras de negócio.
   * Utilize `jqwik` para descobrir falhas de borda por meio de testes baseados em propriedades.
3. **YAGNI (You Aren't Gonna Need It)**:
   * Evite implementar código antecipadamente sem um caso de uso claro associado. Sem "futurismo" de código.
4. **Tratamento de Exceções**:
   * Exceções do domínio ou aplicação não podem vazar diretamente HTTP Status para fora. O `presentation` captura as exceções e mapeia em DTOs estruturados padrão RFC 7807 (Problem Details).

---

## 📦 Dependências Críticas

| Dependência | Versão | Justificativa |
| :--- | :--- | :--- |
| **Java** | `21` | Versão LTS moderna com suporte a Pattern Matching, Records estáveis e melhorias de performance. |
| **Spring Boot** | `3.2.11` | Framework corporativo robusto para bootstrap, injeção de dependência e suporte REST. |
| **JSQLParser** | `4.8` | Biblioteca de parsing de SQL que lê e interpreta DDLs de forma precisa para a extração do schema. |
| **P6Spy Starter** | `1.9.1` | Interceptador JDBC para profiling de SQL queries em tempo de execução, vital para detectar problemas de performance como N+1. |
| **jqwik** | `1.8.5` | Framework de Property-based Testing integrado ao JUnit 5 para validações matemáticas e de integridade dos DDLs analisados. |
| **Flyway** | `3.2` parent | Ferramenta de versionamento e migração de banco de dados PostgreSQL estruturado e resiliente. |
| **FreeMarker** | `2.3` parent starter | Motor de templates para renderização desacoplada do código-fonte das APIs geradas. |

---

## 📄 ADRs (Architecture Decision Records)

### ADR 001: Escolha da Stack e Linguagem (Java 21 + Spring Boot 3.2)
* **Status**: Aprovado
* **Decisão**: Usar Java 21 LTS e Spring Boot 3.2.11 para facilidade de manutenibilidade, records e Virtual Threads.

### ADR 002: Adoção de Arquitetura Limpa (Clean Architecture) Estrita
* **Status**: Aprovado
* **Decisão**: Dividir o sistema em quatro camadas estritas (`domain`, `application`, `infrastructure`, `presentation`) mantendo o motor gerador independente de frameworks ou entrega.

### ADR 003: Uso de P6Spy para Prevenção de Query N+1
* **Status**: Aprovado
* **Decisão**: Utilizar `p6spy-spring-boot-starter` no escopo local para monitorar e evitar queries JPA/Hibernate ineficientes.

### ADR 004: jqwik para Validação de Parser SQL
* **Status**: Aprovado
* **Decisão**: Adicionar `jqwik` para realizar testes baseados em propriedades gerando strings SQL arbitrárias para testar a robustez do parsing.

### ADR 005: Isolamento Estrito do Motor de Templates (FreeMarker)
* **Status**: Aprovado
* **Decisão**: Utilizar exclusivamente templates FreeMarker (`.ftl`) no diretório `src/main/resources/templates/` injetando metadados estruturados.

### ADR 006: Validação de Código Gerado com Stubs Dinâmicos e JavaCompiler
* **Status**: Aprovado
* **Decisão**: Utilizar a Java Compiler API (`javax.tools.JavaCompiler`) e stubs leves das anotações Lombok, MapStruct e Jakarta Jakarta na suite de testes para compilar e validar o código gerado sem poluir dependências de produção.

### ADR 007: Pluralizador e Conversor de Nomenclatura Embutido (YAGNI)
* **Status**: Aprovado
* **Decisão**: Implementar um utilitário nativo puro (`NamingConventionService`) baseado em regras heurísticas de sufixo e tabelas de mapeamento leves.

### ADR 008: Empacotamento ZIP em Memória Independente de Plataforma
* **Status**: Aprovado
* **Decisão**: Criar o `ZipGeneratorService` na infraestrutura utilizando a API nativa `java.util.zip.ZipOutputStream`, forçando normalização absoluta de barras (`/`) e stripping de caminhos iniciais.

### ADR 009: Adoção de SseEmitter em Thread-Pool Separada para Previews Real-time
* **Status**: Aprovado
* **Decisão**: Utilizar `SseEmitter` e desacoplar o processamento da thread original do Tomcat despachando a tarefa assíncrona por meio de `CompletableFuture.runAsync`.

### ADR 010: Fire-and-Forget Seguro com @Async para Auditoria de Geração
* **Status**: Aprovado
* **Decisão**: Habilitar execução assíncrona com `@EnableAsync` e `@Async` para logs de auditoria no PostgreSQL, assegurando downloads instantâneos mesmo se a persistência oscilar.

### ADR 011: Centralização e Padronização de Exceções com RFC 7807 (Problem Details)
* **Status**: Aprovado
* **Decisão**: Utilizar `@RestControllerAdvice` criando o `GlobalExceptionHandler` e serializando as respostas estritamente sob o padrão internacional RFC 7807 (Problem Details).

### ADR 012: Arquitetura AI Enhancement Layer e Silent Fallback (Sprint 04)
* **Status**: Aprovado
* **Decisão**: Desenvolver um gateway altamente protegido com politicas de resiliência robustas do Resilience4j (Retry e Circuit Breaker). Adotar a política de **Silent Fallback** estrita retornando um `EnrichedSchema` neutro para manter 100% de estabilidade sob indisponibilidades.

### ADR 013: Arquitetura do Frontend Playground (Sprint 05)
* **Status**: Aprovado
* **Decisão**: Utilizar Angular 17 Standalone com formulários reativos, comunicação nativa SSE por `EventSource` encapsulados em observables RxJS, downloads do tipo Blob HTTP (ZIP), Tailwind CSS puro e o Monaco Editor offline mapeado localmente via assets.

---

## 🌟 Portfolio Narrative

Como vitrine definitiva de competências para portfólio de engenharia sênior internacional, o APIForge demonstra excelência técnica através das seguintes realizações estratégicas:

1. **Arquitetura Orientada à Resiliência & Fault-Tolerance**: Demonstra a capacidade de integrar modelos de Inteligência Artificial de forma hiper-segura e tolerante a falhas, implementando **Circuit Breakers** e **Retry Policies** com Resilience4j. O padrão de **Silent Fallback** assegura que mesmo sob queda total da API da OpenAI, o fluxo crítico de geração de código continue operando ininterruptamente com 100% de uptime.
2. **Sistemas Reativos & Streaming de Alta Performance**: Utiliza padrões assíncronos modernos de comunicação via **Server-Sent Events (SSE)**. Ao desacoplar processamento por meio de threads apartadas com `SseEmitter` e processar streams de eventos reativos no Angular 17 utilizando RxJS, a aplicação comprova profundo conhecimento em arquitetura orientada a eventos e baixa latência sem causar esgotamento na Thread Pool do Servlet.
3. **Verificação de Compilação & Metaprogramação Avançada**: Apresenta uma abordagem inovadora e matematicamente precisa de qualidade de código ao programar a **Java Compiler API (`javax.tools.JavaCompiler`)** para validar o código compilado em tempo de execução. O uso de **Lightweight Annotation Stubs** no ambiente de testes valida a integridade sintática e de importações do código gerado (incluindo dependências como Lombok, Jakarta e MapStruct) sem acoplar a infraestrutura de produção.
4. **DevOps Moderno & Infraestrutura como Código (IaC)**: Configuração de um pipeline robusto e enxuto com builds **Docker multi-stage**, execução segura por meio de usuário não-root (`spring:spring`), e proxy Nginx configurado especificamente para burlar buffers e timeouts de conexões SSE. Inclui orquestração em lote via **Docker Compose** (com dependências ordenadas por health checks) e arquivos de deploy declarativos para **Render Blueprint** (ligação automática de DB) e **Vercel** (proteção de rotas SPA).
5. **Clean Architecture Estrita & Qualidade de Software**: Separação rigorosa de responsabilidades de negócios em camadas (`domain`, `application`, `infrastructure`, `presentation`) completamente livres de frameworks na sua raiz de domínio. Comprovada por testes robustos que incluem property-based testing com `jqwik` para stress-test de sintaxe de entrada, Testcontainers para bancos de dados reais e WireMock para mockar falhas da web.

---
> Source: [joaogabriel43/APIForge](https://github.com/joaogabriel43/APIForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
