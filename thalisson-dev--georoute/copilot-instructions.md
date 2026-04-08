## georoute

> ﻿# Sipel Logistics Helper

﻿# Sipel Logistics Helper

Este documento serve como guia central para o desenvolvimento e manutenção do projeto Sipel Logistics Helper.

## 1. Visão Geral
O projeto é uma aplicação Full Stack destinada a auxiliar em processos logísticos. O sistema permite o cadastro e consulta de informações de clientes (instalação, conta contrato, geolocalização) e oferece integração com o Google Maps para rotas.

### Tecnologias Principais
*   **Backend:** Java 21, Spring Boot 3.4.2, Spring Data JPA, Flyway, MapStruct, PostgreSQL, Redis, Spring Security (JWT).
*   **Frontend:** Angular 19+ (Standalone Components, Material Design).

## 2. Arquitetura do Backend
O backend segue uma arquitetura em camadas clássica:

1.  **Controllers (web):** Camada REST que recebe as requisições HTTP.
    *   ClientesController: Gerencia operações de CRUD e busca de clientes (com paginação e cache).
    *   MapsController: Gerencia redirecionamentos para serviços de mapa.
    *   AuthenticationController: Gerencia login e emissão de tokens JWT.
2.  **Services (service):** Contém a lógica de negócios.
3.  **Domain (domain):** Entidades JPA e DTOs.
4.  **Repositories (
epository):** Interface com o banco de dados via Spring Data JPA.
5.  **Mappers:** Conversão entre Entidades e DTOs usando MapStruct.
6.  **Security (infra/security):** Configuração de segurança Stateless com JWT e filtros CORS.

### Banco de Dados & Cache
*   **Gerenciamento de Schema:** Flyway (src/main/resources/db/migration).
*   **Banco Principal:** PostgreSQL.
*   **Cache:** Redis (Implementado com chaves compostas para suportar paginação).

## 3. Configuração e Execução (Backend)

### Pré-requisitos
*   JDK 21 instalado.
*   Maven instalado.
*   PostgreSQL rodando.
*   Redis rodando.

### Executando a Aplicação
No diretório raiz do backend:

`ash
# Compilar e baixar dependências
./mvnw clean install

# Rodar a aplicação
./mvnw spring-boot:run
`

## 4. Documentação da API

### Autenticação
*   POST /api/v1/auth/login: Realiza login e retorna token Bearer.
*   POST /api/v1/user/register: Registra novo usuário.

### Clientes
*   POST /api/v1/clientes: Cadastra um novo cliente (Requer Auth).
*   POST /api/v1/clientes/import: Importação via CSV (Async, Requer Auth).
*   GET /api/v1/clientes/instalacao/{id}: Busca por número de instalação (Público, Cacheado).
*   GET /api/v1/clientes/conta-contrato/{id}: Busca paginada por conta contrato (Público, Cacheado).
*   GET /api/v1/clientes/numero-serie/{id}: Busca paginada por número de série (Público, Cacheado).
*   GET /api/v1/clientes/numero-poste/{id}: Busca paginada por número do poste (Público, Cacheado).

### Maps
*   GET /api/v1/maps/redirect: Redireciona para o Google Maps.

### Rotas (RouteController)
*   **POST /api/v1/routes/optimize:** Calcula rota otimizada. Suporta `currentLat` e `currentLon` opcionais. Se não fornecidos, usa a base da equipe.
*   **GET /api/v1/routes/history/{teamId}:** Retorna histórico de execuções.
*   **GET /api/v1/routes/{id}:** Retorna detalhes de uma rota específica (JSON completo).

## 5. Frontend (Angular)

O frontend foi desenvolvido utilizando Angular moderno (Standalone Components).

### Funcionalidades Implementadas
*   **Busca Unificada:** Interface de busca que detecta automaticamente o tipo de dado (Instalação, Conta Contrato, etc).
*   **Paginação:** Integração completa com os endpoints paginados do backend (Pageable).
*   **Visualização:** Exibição dos dados do cliente e link direto para rota no Google Maps.
*   **Planejador de Rotas (Melhorado):**
    *   **Abas:** Separação entre "Nova Rota" e "Histórico".
    *   **Geolocalização:** Opção para usar a localização atual do dispositivo ou a base operacional da equipe como ponto de partida/chegada.
    *   **Feedback Visual:** UI refinada com linha do tempo para os passos da rota.

### Executando o Frontend
No diretório rontend:
`ash
npm install
ng serve
`
Acesse: http://localhost:4200

---
*Gerado pelo Assistente Gemini CLI*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Thalisson-DEV)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Thalisson-DEV)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
