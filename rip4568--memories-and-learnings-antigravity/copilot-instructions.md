## memories-and-learnings-antigravity

> > **LEITURA OBRIGATÓRIA**: Você DEVE ler este arquivo `GEMINI.md` por completo ao iniciar uma nova conversa antes de realizar qualquer tarefa. Isso garante que você esteja alinhado com as instruções mais recentes.

# GEMINI 3.0 Agent Manifest

> [!IMPORTANT]
> **LEITURA OBRIGATÓRIA**: Você DEVE ler este arquivo `GEMINI.md` por completo ao iniciar uma nova conversa antes de realizar qualquer tarefa. Isso garante que você esteja alinhado com as instruções mais recentes.

## Principios
Sempre ler os arquivos de memorys e learnings do projeto e contexto que estiver trabalhando, assim garante de seguir estrtura proposta.

## Regras de Escopo de Busca
- **Escopo Principal**: Restrinja buscas e operações ao diretório `Desktop/work`, onde se concentra a maioria dos projetos.
- **Performance & Exclusão**: **SEMPRE** utilize padrões de exclusão para pastas de dependências e arquivos de build em qualquer ferramenta de busca (`grep_search`, `glob`, `list_directory`). 
  - Pastas a ignorar: `node_modules`, `vendor`, `.git`, `.angular`, `.nx`, `dist`, `out`, `storage`, `temp`.
- **Validação de Subprojeto**: Antes de realizar buscas globais em um repositório monorepo ou com múltiplas pastas (ex: `golembrar-web` vs `golembrar-admin`), valide em qual subdiretório a tarefa deve ser executada para evitar buscas desnecessárias.

## 1. Core Directives
**ALWAYS** read the internal files in the `global_skills` folder to align with the user's coding standards.
- **Location**: `C:\Users\user\.gemini\antigravity\global_skills`

## 2. Organization & Memory

### Projects Memory (`projects_memory/`)
**Goal**: Context preservation per project.
- **Rule**: Create a distinct folder for **each** project inside `projects_memory/`. Inside that folder, create Markdown files for each major component.
- **Naming Convention**: `projects_memory/[ProjectName]/[ProjectName]_[Type].md` (e.g., `projects_memory/Ecommerce/Ecommerce_Backend.md`).
- **Content**:
  - Tech Stack & Libraries
  - Folder Structure & Architecture
  - Design Patterns Used (Repositories, Services, etc.)
  - Key Decisions & Trade-offs

#### 📚 Aprendizados Recentes

> [!IMPORTANT]
> **Consulte sempre o arquivo central de aprendizados (índice global)** antes de iniciar qualquer tarefa para evitar erros de versão ou configuração.

- 📖 **Índice Global de Aprendizados & Bugs**: [LEARNING.md](file:///C:/Users/user/.gemini/antigravity/learning/LEARNING.md)

---
*Nota: O arquivo LEARNING.md serve como índice mestre contendo os links para todos os aprendizados registrados.*

## 3. Path Constants
- **global_skills**: `C:\Users\user\.gemini\antigravity\global_skills`
- **Root**: `C:\Users\user\.gemini\antigravity`
- **Memory**: `C:\Users\user\.gemini\antigravity\projects_memory`
- **Learning**: `C:\Users\user\.gemini\antigravity\learning`

## 4. Language
Sempre me responda em pt-br, tanto ao fazer os planejamentos quanto as tasks.

## 5. Pesquisa & Documentação
**Priorize a busca externa**. Em integrações ou erros recorrentes, utilize o acesso à internet para consultar documentações oficiais e a comunidade. **Não assuma comportamentos nem entre em loops de suposições**; valide incertezas com fontes atualizadas.

## 6. Revisar!
Ao fazer qualquer entrega de codigo, revise os aruquivos editados e garanta que neles não há erros de sintaxe como falta de fechamento de parenteses, chaves ou etc. Verifique se há algum erro de tipagem tambem e corrija.

## 7 Strict Obedience & Checklist Protocol
Para CADA solicitação do usuário, crie um checklist passo-a-passo no `task.md` ANTES de alterar qualquer código.
Nunca marque um item como feito sem verificar explicitamente o arquivo modificado.
Para CADA solicitação do usuario deve ser feito autoanalise tanto da sua solicitação e intenção do pedido quanto do contexto em que tiver trabalhando e entender TODOS os pontos que ele pediu e cumprir com cada uma das suas alterações.

## 8 Clean Comments:
**Não** coloque comentários óbvios no código. O código deve ser autoexplicativo. E se for colocar, coloque em português brasil.

## 9. Ativar Pareamento de Modelos (Gemini 3 Pro & Flash)

Quando essa funcionalidade for solicitada, trabalhe de forma colaborativa prevendo a divisão eficaz do escopo:

- **Gemini 3 Pro**: Foque nas tarefas núcleo de maior complexidade arquitetural (estruturação do código, criação de componentes base e páginas core, além de lógica pesada de negócio).
- **Gemini 3 Flash**: Assuma tarefas de construção repetitiva e reorganização (desenvolvimento de telas complementares utilizando os componentes criados preliminarmente, refatorações simples e design secundário).
- **Auto-Identificação e Hand-off**: Reconheça automaticamente em qual modelo você está operando. Ao encerrar a sua etapa do trabalho, elabore um prompt detalhado e estruturado contendo as pendências, o contexto global resolvido e as interações realizadas (preferencialmente em arquivos markdown auxiliares) garantindo continuidade total para o modelo subsequente assumir o desenvolvimento no mesmo branch/PR.

---
> Source: [Rip4568/memories_and_learnings_antigravity](https://github.com/Rip4568/memories_and_learnings_antigravity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
