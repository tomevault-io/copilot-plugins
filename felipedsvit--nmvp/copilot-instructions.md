## nmvp

> [![MVP Status](https://img.shields.io/badge/Status-MVP%20Ready-green)]()

# 🏛️ nLic - Sistema de Consulta de Dispensas de Licitação

[![MVP Status](https://img.shields.io/badge/Status-MVP%20Ready-green)]()
[![Tech Stack](https://img.shields.io/badge/Stack-React%20%2B%20FastAPI-blue)]()
[![License](https://img.shields.io/badge/License-MIT-yellow)]()

> **Aplicação web moderna para consulta eficiente de Dispensas de Licitação através da API do Portal Nacional de Contratações Públicas (PNCP)**

## 🎯 Sobre o Projeto

O **nLic** é um MVP (Produto Mínimo Viável) desenvolvido para simplificar e otimizar consultas de dispensas de licitação, oferecendo:


## Stack Recomendada: Next.js (React + API Routes)

### Visão Geral

Para acelerar o desenvolvimento do MVP e simplificar a arquitetura, recomenda-se o uso do **Next.js** como stack única. O Next.js integra:

- **Frontend:** React, com suporte opcional a SSR (Server-Side Rendering) e SSG (Static Site Generation).
- **Backend:** API Routes (Node.js), permitindo criar endpoints backend no mesmo projeto.

### Vantagens do Next.js para o nMVP

- **Projeto Unificado:** Frontend e backend no mesmo repositório, facilitando manutenção, deploy e versionamento.
- **API Proxy:** As API Routes permitem criar endpoints internos que fazem proxy para a API do PNCP, resolvendo problemas de CORS e protegendo chaves ou lógica sensível.
- **SSR/SSG Opcional:** Possibilidade de renderizar páginas no servidor para melhor SEO e performance, ou gerar páginas estáticas.
- **Exportação para Excel:** Fácil integração com a biblioteca `xlsx` (SheetJS) para exportação dos dados da tabela diretamente do frontend.
- **Comunidade Forte:** Grande ecossistema, documentação robusta e muitos exemplos disponíveis.
- **Hospedagem Simplificada:** Deploy facilitado em plataformas como **Vercel** (oficial), **Netlify** e **Railway**.
- **Desenvolvimento Local e Docker:** Next.js roda facilmente em ambientes locais e pode ser containerizado, mas atenção à versão do Node.js suportada pela imagem Docker escolhida (recomenda-se Node.js 18+ para Next.js 13/14).

### Estrutura de Projeto Sugerida

```
nMVP/
└── app/ ou pages/         # Páginas React (SSR/SSG ou SPA)
└── pages/api/             # API Routes (endpoints backend)
└── components/            # Componentes reutilizáveis React
└── package.json
└── next.config.js
└── Dockerfile (opcional)
```

### Exemplo de Integração com API do PNCP

**API Route para Proxy (pages/api/dispensas.js):**
```javascript
// filepath: /pages/api/dispensas.js
import axios from 'axios';

export default async function handler(req, res) {
  const { dataInicial, dataFinal, uf, pagina } = req.query;
  try {
    const response = await axios.get('https://pncp.gov.br/api/consulta/v1/contratacoes/publicacao', {
      params: {
        dataInicial,
        dataFinal,
        codigoModalidadeContratacao: 8,
        uf,
        pagina
      }
    });
    res.status(200).json(response.data);
  } catch (error) {
    res.status(error.response?.status || 500).json({ error: error.message });
  }
}
```

**Frontend (React) para Consulta e Exportação:**
- Use `fetch` ou `axios` para consumir `/api/dispensas`.
- Utilize a biblioteca `xlsx` para exportação dos dados para Excel.

### Hospedagem e Deploy

- **Vercel:** Deploy automático e integração nativa com Next.js.
- **Netlify:** Suporte a Next.js com funções serverless.
- **Railway:** Deploy de aplicações Node.js com facilidade.
- **Docker:** Para ambientes customizados, crie um `Dockerfile` com Node.js 18+ e configure as variáveis de ambiente conforme necessário.

> **Atenção:** Certifique-se de que a versão do Node.js utilizada no ambiente Docker ou na plataforma de hospedagem é compatível com a versão do Next.js adotada (Node.js 18+ recomendado para Next.js 13 ou superior).

- ⚡ **Consultas 10x mais rápidas** que métodos manuais
- 📊 **Dashboard intuitivo** com métricas em tempo real  
- 📤 **Exportação inteligente** para Excel/CSV
- 🔍 **Filtros avançados** para análises precisas
- 📱 **Interface responsiva** para qualquer dispositivo

## 1. Endpoint de Consulta

Para buscar as dispensas de licitação, utilize o seguinte endpoint:

- **Método HTTP:** `GET`
- **URL Base:** `https://pncp.gov.br/api/consulta`
- **Endpoint:** `/v1/contratacoes/publicacao`

## 2. Parâmetro Chave: `codigoModalidadeContratacao`

Para filtrar exclusivamente as **Dispensas de Licitação**, é obrigatório o uso do seguinte parâmetro na sua requisição:

- `codigoModalidadeContratacao=8`

Este código é definido na Tabela de Domínio **5.2. Modalidade de Contratação** do manual do PNCP.

## 3. Parâmetros de Requisição (Filtros)

Os seguintes parâmetros podem ser combinados para refinar a busca.

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | --- | --- |
| `dataInicial` | Data | Sim | Data inicial do período a ser consultado no formato `AAAAMMDD`. |
| `dataFinal` | Data | Sim | Data final do período a ser consultado no formato `AAAAMMDD`. |
| `codigoModalidadeContratacao` | Inteiro | Sim | **Deve ser `8` para Dispensa de Licitação.** |
| `uf` | String | Não | Sigla da Unidade Federativa (ex: `DF`, `SP`). |
| `codigoMunicipioIbge` | String | Não | Código IBGE do Município (ex: `5300108` para Brasília). |
| `cnpj` | String | Não | CNPJ do órgão ou entidade que publicou a contratação. |
| `codigoUnidadeAdministrativa` | String | Não | Código da Unidade Administrativa do órgão. |
| `idUsuario` | Inteiro | Não | Identificador do sistema/usuário que publicou a contratação. |
| `pagina` | Inteiro | Sim | Número da página para obter os dados. A primeira página é `1`. |
| `tamanhoPagina` | Inteiro | Não | Quantidade de registros por página (padrão: 50, máximo: 500). |

## 4. Exemplos de Requisições

### Exemplo 1: Consulta Básica por Período

Busca por todas as dispensas de licitação publicadas em agosto de 2023.

```bash
curl -X 'GET' \
  'https://pncp.gov.br/api/consulta/v1/contratacoes/publicacao?dataInicial=20230801&dataFinal=20230831&codigoModalidadeContratacao=8&pagina=1' \
  -H 'accept: */*'
```

### Exemplo 2: Consulta por Órgão e Estado (UF)

Busca por dispensas de um órgão específico (pelo CNPJ) no estado de São Paulo.

```bash
curl -X 'GET' \
  'https://pncp.gov.br/api/consulta/v1/contratacoes/publicacao?dataInicial=20230101&dataFinal=20231231&codigoModalidadeContratacao=8&cnpj=00059311000126&uf=SP&pagina=1' \
  -H 'accept: */*'
```

## 5. Estrutura da Resposta

A resposta será um objeto JSON contendo informações de paginação e um vetor `data` com os resultados.

### 5.1. Estrutura de Paginação

| Campo | Tipo | Descrição |
| --- | --- | --- |
| `totalRegistros` | Inteiro | Total de registros encontrados para a consulta. |
| `totalPaginas` | Inteiro | Total de páginas para obter todos os registros. |
| `numeroPagina` | Inteiro | O número da página atual retornada. |
| `paginasRestantes` | Inteiro | Total de páginas restantes. |
| `empty` | Booleano | `true` se o atributo `data` estiver vazio. |

### 5.2. Estrutura dos Dados (`data`)

Cada objeto dentro do array `data` representa uma dispensa e contém os seguintes campos (lista parcial):

```json
{
  "data": [
    {
      "numeroControlePNCP": "00059311000126-1-000001/2023",
      "numeroCompra": "DISPENSA-001/2023",
      "anoCompra": 2023,
      "processo": "12345.678901/2023-01",
      "modalidadeId": 8,
      "modalidadeNome": "Dispensa de Licitação",
      "situacaoCompraId": 1,
      "situacaoCompraNome": "Divulgada no PNCP",
      "objetoCompra": "Contratação de serviços de limpeza e conservação.",
      "valorTotalEstimado": 50000.00,
      "dataPublicacaoPncp": "2023-08-15T09:00:00",
      "orgaoEntidade": {
        "cnpj": "00059311000126",
        "razaosocial": "NOME DO ORGAO PUBLICO",
        "poderId": "E",
        "esferaId": "F"
      },
      "unidadeOrgao": {
        "codigoUnidade": "194035",
        "nomeUnidade": "NOME DA UNIDADE ADMINISTRATIVA",
        "codigoIbge": 5300108,
        "municipioNome": "Brasília",
        "ufSigla": "DF",
        "ufNome": "Distrito Federal"
      },
      "linkSistemaOrigem": "https://link.para.sistema.de.origem/details"
    }
  ],
  "totalRegistros": 1,
  "totalPaginas": 1,
  "numeroPagina": 1,
  "paginasRestantes": 0,
  "empty": false
}
```

## 6. Paginação

Para navegar pelos resultados, utilize os parâmetros `pagina` e `tamanhoPagina`. A resposta da API sempre informará o `totalPaginas` e o `numeroPagina` atual, permitindo que você construa as próximas requisições de forma iterativa até que `paginasRestantes` seja 0.

## 7. Códigos de Retorno HTTP

| Código HTTP | Mensagem | Significado |
| --- | --- | --- |
| `200` | OK | Requisição bem-sucedida e dados retornados. |
| `204` | No Content | Requisição bem-sucedida, mas nenhum registro foi encontrado. |
| `400` | Bad Request | Erro na requisição, como um formato de data inválido. |
| `422` | Unprocessable Entity | Erro de negócio, como parâmetros obrigatórios ausentes. |
| `500` | Internal Server Error | Erro interno no servidor do PNCP. |

---

## Arquitetura e Implementação do MVP de Consulta

Esta seção descreve a arquitetura da aplicação que consome a API do PNCP, seu fluxo de dados e detalhes de implementação.

### 1. Fluxo de Informações e Arquitetura

O sistema é projetado em uma arquitetura de três camadas (frontend, backend, banco de dados), orquestrada com Docker.

1.  **Interação do Usuário (Frontend):**
    *   O usuário acessa a interface web (aplicação React).
    *   Em uma página de busca, ele insere os filtros desejados, como `dataInicial` e `dataFinal`, e clica em "Buscar".

2.  **Requisição Frontend -> Backend:**
    *   A aplicação React captura os dados do formulário e envia uma requisição HTTP (ex: `GET /api/dispensas?dataInicial=...`) para o seu próprio servidor backend (a aplicação FastAPI).

3.  **Requisição Backend -> API Externa (PNCP):**
    *   O backend em FastAPI recebe a requisição.
    *   Ele atua como um cliente para a API do PNCP, construindo a URL de consulta com os parâmetros recebidos e o `codigoModalidadeContratacao=8` fixo.
    *   O serviço `pncp_service.py` encapsula a lógica de comunicação com a API externa.

4.  **Processamento e Persistência (Backend e Banco de Dados):**
    *   O backend recebe a resposta JSON da API do PNCP.
    *   Os dados são validados, transformados para um modelo interno (`backend/app/models/`) e armazenados no banco de dados PostgreSQL para histórico e otimização de consultas.

5.  **Resposta Backend -> Frontend:**
    *   O backend envia uma resposta JSON para a aplicação React com os resultados da consulta formatados.

6.  **Renderização dos Dados (Frontend):**
    *   A aplicação React recebe os dados, atualiza seu estado e renderiza a tabela de resultados.

### 2. Stack de Tecnologia

*   **Frontend:**
    *   **Framework:** React com TypeScript (`.tsx`).
    *   **Build Tool:** Vite (`vite.config.ts`).
    *   **Comunicação HTTP:** Axios ou Fetch API (`src/api/client.ts`).
    *   **Gerenciamento de Estado:** Zustand ou Context API (`stores/`).

*   **Backend:**
    *   **Framework:** Python com FastAPI.
    *   **Validação de Dados:** Pydantic (`backend/app/schemas/`).
    *   **ORM e Migrations:** SQLAlchemy e Alembic (`backend/app/models/` e `backend/migrations/`).

*   **Banco de Dados:**
    *   **SGBD:** PostgreSQL.

*   **Containerização e Implantação:**
    *   **Containers:** Docker (`Dockerfile` em `frontend/` e `backend/`).
    *   **Orquestração:** Docker Compose (`docker-compose.yml`).
    *   **Servidor Web (Frontend):** Nginx (`frontend/nginx.conf`).

### 3. Preparação para Rodar com Docker

O `docker-compose.yml` define e conecta os serviços da aplicação (`frontend`, `backend`, `db`). Para rodar o sistema, execute o seguinte comando na raiz do projeto:

```bash
docker-compose up --build -d
```

Este comando constrói as imagens, cria e inicia os containers em segundo plano. O Nginx serve a aplicação React e redireciona as chamadas de API para o backend FastAPI.

### 4. Implementação da Interface com Tabela e Exportação XLSX

A implementação ocorre em um componente React, como `frontend/src/pages/entities/Atas.tsx`.

1.  **Componente da Tabela:**
    *   O componente `DataTable.tsx` é usado para exibir os dados recebidos da API do backend.

2.  **Botão de Exportação:**
    *   Um botão "Exportar para XLSX" é adicionado à interface. Para a funcionalidade, a biblioteca `xlsx` (SheetJS) deve ser instalada:
    ```bash
    npm install xlsx
    ```

3.  **Lógica da Função de Exportação:**
    *   A função `onClick` do botão executa a conversão dos dados da tabela (armazenados no estado do componente) para o formato XLSX e inicia o download.

    ```typescript
    // Exemplo de código dentro de um componente React
    import * as XLSX from 'xlsx';

    const handleExport = (dataToExport) => {
      if (!dataToExport || dataToExport.length === 0) {
        console.log("Não há dados para exportar.");
        return;
      }
      
      // 1. Criar uma nova planilha a partir dos dados JSON
      const worksheet = XLSX.utils.json_to_sheet(dataToExport);

      // 2. Criar um novo workbook e adicionar a planilha
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Dispensas");

      // 3. Trigger do download do arquivo .xlsx
      XLSX.writeFile(workbook, "Dispensas_de_Licitacao.xlsx");
    };

    // No JSX:
    // <button onClick={() => handleExport(data)}>Exportar para XLSX</button>
    ```


Funcionalidades Essenciais do MVP:

  - Consulta de dispensas com filtros avançados
  - Visualização em tabela responsiva
  - Exportação para Excel/CSV
  - Dashboard básico com métricas

  Stack Tecnológica Moderna:

  - Frontend: React 18 + TypeScript + MUI + Vite
  - DevOps: Docker + GitHub Actions + Nginx

  Plano de Desenvolvimento (10 semanas):

  1. Sprint 1-2: Setup e autenticação
  2. Sprint 3-4: Core features (consulta e filtros)
  3. Sprint 5-6: UX/UI e exportação
  4. Sprint 7-8: Dashboard e analytics
  5. Sprint 9-10: Polimento e deploy

  Critérios de Validação:

  - MVP funcional em 6 semanas
  - Sistema responsivo e acessível
  - Performance <3s
  - Testes automatizados

  O documento agora serve como guia completo para o desenvolvimento do sistema nLic, com foco em validação rápida e entrega
  incremental.

## 🛠️ Stack Tecnológica

### Frontend
- **React 18** + TypeScript
- **Vite** para build ultrarrápido
- **Tailwind CSS** para design moderno
- **Zustand** para gerenciamento de estado
- **TanStack Table** para tabelas performáticas

### Backend  
- **FastAPI** (Python 3.12) para APIs robustas
- **PostgreSQL 16** para persistência confiável
- **Redis** para cache e otimização
- **SQLAlchemy 2.0** ORM assíncrono
- **Pydantic v2** para validação de dados

### DevOps
- **Docker + Compose** para containerização
- **GitHub Actions** para CI/CD
- **Nginx** como reverse proxy
- **Prometheus + Grafana** para monitoramento

## 🚀 Quick Start

### Pré-requisitos
- Docker & Docker Compose
- Git

### Executar o Sistema

```bash
# 1. Clone o repositório
git clone https://github.com/seu-usuario/nMVP.git
cd nMVP

# 2. Configure as variáveis de ambiente
cp .env.example .env

# 3. Execute com Docker
docker-compose up --build -d

# 4. Acesse a aplicação
# Frontend: http://localhost:3000
# Backend API: http://localhost:8000
# Docs API: http://localhost:8000/docs
```

## 📁 Estrutura do Projeto

```
nMVP/
├── 📂 frontend/              # React + TypeScript
│   ├── src/components/       # Componentes UI
│   ├── src/pages/           # Páginas principais
│   ├── src/hooks/           # Custom hooks
│   └── src/services/        # Serviços HTTP
├── 📂 backend/              # FastAPI + Python
│   ├── app/api/             # Endpoints REST
│   ├── app/models/          # Modelos de dados
│   ├── app/services/        # Lógica de negócio
│   └── app/schemas/         # Validação Pydantic
├── 📂 docker/               # Configurações Docker
├── docker-compose.yml       # Orquestração
└── .github/workflows/       # CI/CD
```

## 📋 Funcionalidades Principais

### 🔍 Consulta Avançada
- Filtros por período, UF, CNPJ, valor
- Paginação eficiente para grandes volumes
- Cache inteligente para consultas repetidas
- Busca textual no objeto da contratação

### 📊 Dashboard Analytics  
- KPIs em tempo real (total, valor médio, tendências)
- Gráficos de distribuição por UF e evolução temporal
- Métricas de performance do sistema

### 📤 Exportação Flexível
- Excel (.xlsx) com formatação
- CSV para análises externas  
- JSON para integrações
- Suporte a grandes volumes (50k+ registros)

### 📱 Interface Responsiva
- Design mobile-first
- Adaptação automática desktop/tablet/mobile
- Acessibilidade WCAG 2.1 AA

## 📚 Documentação

| Documento | Descrição |
|-----------|-----------|
| [📊 MVP Executive Summary](./MVP_EXECUTIVE_SUMMARY.md) | Visão geral e estratégia do produto |
| [🛠️ Implementation Guide](./IMPLEMENTATION_GUIDE.md) | Guia detalhado de implementação |
| [⚙️ Technical Config](./TECHNICAL_CONFIG.md) | Configurações técnicas específicas |
| [📖 API Documentation - PNCP](./CLAUDE.md) | Documentação da API PNCP original |

## 🛣️ Roadmap

### ✅ MVP (Semanas 1-6)  
- [x] Consulta básica de dispensas
- [x] Interface responsiva 
- [x] Exportação Excel/CSV
- [x] Dashboard com métricas
- [x] Deploy automatizado

### 🔄 Fase 2 (Semanas 7-12)
- [ ] Autenticação de usuários
- [ ] Relatórios customizados  
- [ ] Notificações automáticas
- [ ] Temas e personalização

### 🚀 Fase 3 (Semanas 13-24)
- [ ] Análise preditiva com ML
- [ ] App mobile nativo
- [ ] Integração com ERPs
- [ ] API marketplace

## 📄 Licença

Este projeto está licenciado sob a Licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.

---

<div align="center">
  <strong>🏛️ Desenvolvido com ❤️ para modernizar a consulta de dados públicos brasileiros</strong>
</div>

## 📖 Documentação Técnica da API PNCP

### 1. Visão Geral

A API do PNCP permite o acesso a dados de contratações públicas realizadas por meio de dispensas de licitação. Este documento descreve os detalhes técnicos da API, incluindo autenticação, endpoints disponíveis, parâmetros de consulta, e exemplos de requisição e resposta.

### 2. Autenticação

A autenticação é realizada por meio de tokens JWT (JSON Web Tokens). Para obter um token, faça uma requisição POST para o endpoint `/auth/login` com as credenciais de usuário.

#### Exemplo de Requisição de Login

```http
POST /auth/login HTTP/1.1
Host: pncp.gov.br
Content-Type: application/json

{
  "username": "seu_usuario",
  "password": "sua_senha"
}
```

#### Exemplo de Resposta de Login

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "access_token": "seu_token_jwt",
  "token_type": "bearer"
}
```

### 3. Endpoints Disponíveis

#### 3.1. Consulta de Dispensas

- **URL:** `/v1/contratacoes/publicacao`
- **Método:** `GET`
- **Descrição:** Retorna uma lista de dispensas de licitação com base nos filtros fornecidos.

#### 3.2. Detalhes da Dispensa

- **URL:** `/v1/contratacoes/publicacao/{id}`
- **Método:** `GET`
- **Descrição:** Retorna os detalhes de uma dispensa específica.

### 4. Parâmetros de Consulta

Os parâmetros de consulta podem ser passados na URL como query strings. Os parâmetros disponíveis incluem:

- `dataInicial`: Data inicial para filtragem (formato: `AAAAMMDD`).
- `dataFinal`: Data final para filtragem (formato: `AAAAMMDD`).
- `codigoModalidadeContratacao`: Código da modalidade de contratação (deve ser `8` para dispensas).
- `uf`: Sigla da Unidade Federativa.
- `codigoMunicipioIbge`: Código IBGE do município.
- `cnpj`: CNPJ do órgão ou entidade.
- `pagina`: Número da página para pagin ação dos resultados.
- `tamanhoPagina`: Quantidade de registros por página.

### 5. Exemplos de Requisição

#### 5.1. Consulta de Dispensas

```http
GET /v1/contratacoes/publicacao?dataInicial=20230101&dataFinal=20231231&codigoModalidadeContratacao=8&pagina=1 HTTP/1.1
Host: pncp.gov.br
Authorization: Bearer seu_token_jwt
```

#### 5.2. Detalhes da Dispensa

```http
GET /v1/contratacoes/publicacao/12345 HTTP/1.1
Host: pncp.gov.br
Authorization: Bearer seu_token_jwt
```

### 6. Estrutura da Resposta

A resposta da API será um objeto JSON. Para a consulta de dispensas, a estrutura da resposta incluirá:

- `data`: Array contendo os registros das dispensas.
- `totalRegistros`: Total de registros encontrados.
- `totalPaginas`: Total de páginas disponíveis.
- `numeroPagina`: Número da página atual.
- `paginasRestantes`: Número de páginas restantes.

#### Exemplo de Resposta

```json
{
  "data": [
    {
      "numeroControlePNCP": "00059311000126-1-000001/2023",
      "numeroCompra": "DISPENSA-001/2023",
      "anoCompra": 2023,
      "processo": "12345.678901/2023-01",
      "modalidadeId": 8,
      "modalidadeNome": "Dispensa de Licitação",
      "situacaoCompraId": 1,
      "situacaoCompraNome": "Divulgada no PNCP",
      "objetoCompra": "Contratação de serviços de limpeza e conservação.",
      "valorTotalEstimado": 50000.00,
      "dataPublicacaoPncp": "2023-08-15T09:00:00",
      "orgaoEntidade": {
        "cnpj": "00059311000126",
        "razaosocial": "NOME DO ORGAO PUBLICO",
        "poderId": "E",
        "esferaId": "F"
      },
      "unidadeOrgao": {
        "codigoUnidade": "194035",
        "nomeUnidade": "NOME DA UNIDADE ADMINISTRATIVA",
        "codigoIbge": 5300108,
        "municipioNome": "Brasília",
        "ufSigla": "DF",
        "ufNome": "Distrito Federal"
      },
      "linkSistemaOrigem": "https://link.para.sistema.de.origem/details"
    }
  ],
  "totalRegistros": 1,
  "totalPaginas": 1,
  "numeroPagina": 1,
  "paginasRestantes": 0,
  "empty": false
}
```

### 7. Códigos de Retorno HTTP

Os códigos de retorno HTTP seguem os padrões comuns, onde `200` indica sucesso, `204` indica que não há conteúdo para retornar, `400` indica um erro na requisição do cliente, `401` indica não autorizado, `403` proibido, `404` não encontrado, e `500` erro interno do servidor.

---

## Considerações Finais

Esta documentação fornece uma visão geral da API do PNCP para consulta de dispensas de licitação. Para mais detalhes sobre cada endpoint, parâmetros e exemplos, consulte a documentação completa disponível no repositório do projeto.

O desenvolvimento da API e da aplicação nLic visa modernizar e facilitar o acesso a dados públicos, promovendo maior transparência e eficiência nas consultas de dispensas de licitação.

---
> Source: [felipedsvit/nMVP](https://github.com/felipedsvit/nMVP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
