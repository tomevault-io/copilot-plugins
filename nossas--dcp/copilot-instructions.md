## dcp

> Este documento fornece uma visão geral completa do projeto, sua arquitetura, tecnologias, e como iniciar o desenvolvimento.

# GEMINI.md - Análise do Projeto "DCP"

Este documento fornece uma visão geral completa do projeto, sua arquitetura, tecnologias, e como iniciar o desenvolvimento.

## Visão Geral do Projeto

Trata-se de um projeto WordPress customizado para a plataforma DCP (possivelmente Defesa Civil). A aplicação é construída sobre um tema e um plugin customizados que juntos fornecem funcionalidades para monitoramento e gerenciamento de riscos, ações, e informações de apoio à comunidade.

A arquitetura é bem definida, separando as responsabilidades entre a lógica de dados (plugin) e a camada de apresentação (tema). O projeto é totalmente containerizado com Docker, facilitando a configuração do ambiente de desenvolvimento local.

## Principais Funcionalidades

A análise do código revelou um sistema robusto com as seguintes funcionalidades centrais:

*   **API REST Customizada:** O `dcp-plugin` expõe uma série de endpoints públicos (`/dcp/v1/*`) para consumir dados sobre:
    *   **Riscos:** Listagem e resumo de situações de risco reportadas.
    *   **Abrigos:** Lista de locais seguros.
    *   **Recomendações (Dicas):** Conselhos de segurança para a população.
    *   **Contatos de Emergência:** Lista de telefones úteis.
    *   **Situação da Região:** Informações atualizadas sobre o nível de risco local.

*   **Painel de Gerenciamento (Frontend):** O tema implementa uma área de dashboard acessível em `/dashboard` para usuários com a permissão "Agente DCP". Este painel permite gerenciar as diferentes seções do sistema (riscos, ações, etc.) através de uma interface amigável, sem a necessidade de acessar o painel de administração padrão do WordPress.

*   **Tipos de Conteúdo Customizados:** A plataforma parece girar em torno de CPTs como `risco`, `acao`, `apoio`, `recomendacao`, gerenciados pelo plugin [Pods](https://pods.io/).

*   **Integração via Webhook:** O sistema possui um webhook para receber e atualizar a "Situação Atual" a partir de fontes de dados externas.

## Tecnologias

*   **Backend:** PHP, WordPress
*   **Frontend:** JavaScript, SCSS, Alpine.js, Splide.js (para sliders)
*   **Ferramentas de Build:** Node.js, npm, Laravel Mix
*   **Ambiente de Desenvolvimento:** Docker, Docker Compose
*   **Banco de Dados:** MariaDB

## Estrutura do Projeto

*   `themes/dcp`: Diretório do tema principal.
    *   `assets/`: Contém os arquivos fontes de SCSS e JavaScript.
    *   `dist/`: Contém os arquivos compilados e prontos para uso.
    *   `library/`: Organiza a lógica do tema em módulos. Destaques:
        *   `dashboard.php`: Orquestra o painel de gerenciamento do frontend.
        *   `frontend_auth.php`: Cuida da autenticação customizada.
        *   `blocks/`: Registra blocos Gutenberg customizados.
        *   `api/`: Contém arquivos relacionados a customizações na API.
    *   `template-parts/dashboard/`: Contém os arquivos de template para as diferentes seções do painel.
    *   `functions.php`: Ponto de entrada principal do tema.
    *   `package.json`: Define as dependências Node.js.
*   `plugins/dcp-plugin`: Plugin customizado do projeto.
    *   `dcp-plugin.php`: Arquivo principal que define os CPTs (via Pods), endpoints da API e webhooks.
*   `docker-compose.yml`: Define os serviços Docker (WordPress, MariaDB, etc).
*   `dev-scripts/`: Contém scripts úteis para tarefas de desenvolvimento.

## Build e Execução

1.  **Instalar Dependências:**
    *   Certifique-se de ter Docker, Docker Compose, Node.js e npm instalados.
    *   Navegue até o diretório do tema (`themes/dcp`) e execute `npm install` para instalar as dependências de frontend.

2.  **Iniciar o Ambiente de Desenvolvimento:**
    *   Na raiz do projeto, execute `docker-compose up`.
    *   O site estará disponível em `http://localhost`.

3.  **Compilar Assets:**
    *   Navegue até o diretório do tema (`themes/dcp`).
    *   Execute `npm run watch` para recompilar os assets automaticamente ao detectar alterações nos arquivos.
    *   Execute `npm run production` para gerar a versão final dos assets para produção.

## Convenções de Desenvolvimento

*   **Estilo de Código:** O código PHP segue os padrões do WordPress. O código JavaScript e SCSS segue uma abordagem moderna e modular.
*   **Git Flow:** O projeto utiliza um fluxo de trabalho com a branch `develop` para desenvolvimento e `main` para produção.
*   **Submódulos:** Alguns plugins são gerenciados como submódulos do Git.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nossas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nossas)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
