## tabuanimada

> Este arquivo fornece contexto para o agente Gemini sobre o projeto.

# Gemini Context

Este arquivo fornece contexto para o agente Gemini sobre o projeto.

## Visão Geral do Projeto

O projeto é um jogo web de matemática focado em praticar tabuada. Os usuários podem configurar uma partida, jogar respondendo a uma série de cálculos e, ao final, ter seu resultado salvo em um ranking. A interação com o banco de dados ocorre apenas ao final de uma partida completa.

## Tecnologias e Convenções

- **Framework:** Next.js (com Pages Router)
- **Gerenciamento de Estado:** Zustand
- **Biblioteca de UI:** HeroUI (https://www.heroui.com)
- **Ícones:** React-Icons (https://react-icons.github.io/react-icons/)
- **Banco de Dados:** Supabase (SQL)
- **Linguagem:** JavaScript

### Padrões de Código

- **TypeScript:** Não será utilizado.
- **Ponto e vírgula:** O código será escrito sem ponto e vírgula (`;`).
- **Aspas:** Preferência por aspas duplas (`"`).
- **Funções:** Dar preferência a Arrow Functions (`const minhaFuncao = () => {}`). Nos casos de arquivos com uma única função usar (`export default function`)

## Estrutura e Funcionalidades

### Páginas (Rotas)

- `/`: **Home**
- `/ranking`: **Histórico**
- `/config`: **Configurações da Partida**
- `/play`: **Tela da Partida**

### Detalhes das Telas

- **Home (`/`)**
  - Título do projeto.
  - Botão "Jogar" (leva para `/config`).
  - Botão "Ranking" (leva para `/ranking`).

- **Histórico (`/ranking`)**
  - Tabela com o histórico de partidas.
  - Botão para voltar à Home.

- **Configurações (`/config`)**
  - Input para "Nome do jogador".
  - Seleção de "Modo de jogo" (Tabuada e Multiplicação).
  - Input para "Tempo de cada resposta".
  - Input para "Número de perguntas".
  - **Opções de Frequência:**
    - Checkbox para excluir o primeiro termo da multiplicação.
    - Slider para configurar a porcentagem de aparição do segundo termo.
  - Texto descritivo da configuração atual.
  - Botão "Iniciar" (leva para `/play`).
  - Botão "Voltar".

- **Partida (`/play`)**
  - Contador regressivo para iniciar.
  - Emoji de estado que reflete a porcentagem de acertos.
  - Exibição do cálculo da rodada (ex: `A x B = ___`).
  - Contador de progresso das perguntas (ex: `10/20`).
  - Teclado virtual para input da resposta (dígitos 0-9, apagar, confirmar).
  - Botão "Sair" para cancelar a partida e retornar à Home.
  - **Componentes de Feedback:**
    - **Durante a partida:** Após cada resposta, um componente mostra "Acertou!" ou "Errou!", junto com a contagem de acertos e erros.
    - **Final da partida:** Um componente de resumo exibe a porcentagem de acertos/erros e um botão "Concluir" para salvar o resultado.

## Escopo e Limitações

- **Modo Multiplicação:** O desenvolvimento deste modo não faz parte do escopo atual.
- **Persistência de Dados:** Os dados da partida só são enviados para o Supabase quando o jogador clica em "Concluir" na tela de resumo. Partidas abandonadas não são salvas.
- **Testes:** Serão adicionados futuramente utilizando Vitest.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hahayz77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
