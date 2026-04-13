## termo-gemini-cli

> Este documento serve como uma documentação para o projeto Termo, um clone do popular jogo de adivinhação de palavras Wordle/Termo, desenvolvido com Next.js.

# Termo - O Jogo de Palavras

Este documento serve como uma documentação para o projeto Termo, um clone do popular jogo de adivinhação de palavras Wordle/Termo, desenvolvido com Next.js.

## Visão Geral

O jogo desafia o jogador a adivinhar uma palavra secreta de 5 letras em português em até 6 tentativas. Após cada tentativa, o jogo fornece feedback visual sobre as letras, indicando se estão corretas, presentes na palavra (mas na posição errada) ou ausentes.

## Como Jogar

1.  **Adivinhe a Palavra:** Digite uma palavra de 5 letras e pressione Enter.
2.  **Navegação com Setas:** Use as setas esquerda e direita do teclado para mover o cursor entre as posições da palavra atual.
3.  **Feedback Visual:**
    *   **Verde:** A letra está correta e na posição certa.
    *   **Amarelo:** A letra está na palavra, mas na posição errada.
    *   **Cinza:** A letra não está na palavra.
4.  **Continue Tentando:** Use o feedback para refinar suas próximas tentativas. Você tem 6 chances para adivinhar a palavra.
5.  **Novo Jogo:** Após o término do jogo (vitória ou derrota), um botão "Novo Jogo" aparecerá para reiniciar o jogo com uma nova palavra.
6.  **Modal "Como Jogar":** Clique no botão "Como Jogar" para ver as instruções detalhadas e exemplos visuais do feedback das letras.

## Funcionalidades Principais

*   **Seleção de Palavra Aleatória:** A palavra secreta é escolhida aleatoriamente de uma lista de palavras válidas.
*   **Validação de Palavras:** Apenas palavras válidas (existentes na lista) podem ser submetidas como tentativas.
*   **Feedback Visual Detalhado:** O sistema de cores fornece feedback preciso, considerando a frequência das letras na palavra secreta para evitar contagens duplicadas incorretas.
*   **Teclado Interativo:** Um teclado virtual na tela permite a entrada de letras e reflete o status das letras (verde, amarelo, cinza) para auxiliar o jogador. A funcionalidade de digitação foi aprimorada para maior estabilidade.
*   **Navegação por Setas:** Permite ao usuário mover o cursor entre as células de entrada usando as setas do teclado para editar a palavra atual.
*   **API de Palavras:** As palavras válidas são carregadas dinamicamente de um endpoint de API, facilitando a atualização e expansão da lista de palavras sem a necessidade de recompilar o frontend.
*   **Reiniciar Jogo:** Um botão "Novo Jogo" permite reiniciar o jogo a qualquer momento após o término de uma partida.
*   **Modal de Instruções:** Um modal "Como Jogar" fornece instruções claras e exemplos visuais para novos jogadores.

## Arquitetura do Projeto

O projeto segue uma estrutura organizada com base nas boas práticas de desenvolvimento Next.js e React:

*   `src/app/`: Contém as rotas principais da aplicação, incluindo a página inicial (`page.tsx`) e o endpoint da API (`api/words/route.ts`).
*   `src/components/`: Armazena os componentes React reutilizáveis.
    *   `src/components/game/`: Componentes específicos do jogo, como `Game.tsx` (lógica principal), `Grid.tsx` (grade de tentativas), `Keyboard.tsx` (teclado virtual) e `HowToPlayModal.tsx` (modal de instruções).
*   `src/lib/`: Contém funções utilitárias e lógicas de negócio.
    *   `src/lib/gameUtils.ts`: Funções auxiliares para a lógica do jogo, como o cálculo do status das letras.

## Como Rodar o Projeto

Para executar o jogo em seu ambiente local:

1.  **Instale as dependências:**
    ```bash
    npm install
    # ou
    yarn install
    ```
2.  **Inicie o servidor de desenvolvimento:**
    ```bash
    npm run dev
    # ou
    yarn dev
    ```
3.  **Acesse no navegador:** Abra seu navegador e vá para `http://localhost:3000`.

## Histórico de Alterações (Gerenciado pelo Gemini CLI)

*   **23 de julho de 2025:** Criação inicial do projeto Termo. Implementação da estrutura básica, componentes de Grid e Keyboard, e lógica de jogo. Configuração da API para buscar palavras e correção do layout do teclado. Ajuste do `html lang` para `pt-br` e do título da página. Correção da lógica de coloração de letras duplicadas na grade e no teclado. Adicionada funcionalidade de navegação com setas e destaque visual do cursor. Adicionada funcionalidade de "Novo Jogo" para reiniciar a partida. Adicionado modal "Como Jogar" com instruções e botão de acesso. Corrigido problema de digitação no teclado, garantindo que as teclas funcionem corretamente.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zecacorreia)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zecacorreia)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
