## qualidade-codigo

> Princípios gerais de código limpo, testes e controle de versão. Aplicar ao implementar ou revisar código em qualquer linguagem ou stack do repositório.


# Qualidade e código limpo (agnóstico de stack)

Ao implementar ou refatorar código, seguir estes princípios. Para **UI, Next.js, Tailwind e componentes**, complementar com `.cursor/rules/boas-praticas-frontend.mdc` quando existir.

## Constantes em vez de números mágicos

- Substituir valores fixos no código por constantes nomeadas.
- Usar nomes de constantes que expliquem a finalidade do valor.
- Manter constantes no topo do arquivo ou em arquivo dedicado.

## Nomes significativos

- Variáveis, funções e classes devem revelar sua finalidade.
- Nomes devem explicar por que algo existe e como é usado.
- Evitar abreviações, salvo as universalmente entendidas.

## Comentários inteligentes

- Não comentar o que o código faz — tornar o código autoexplicativo.
- Usar comentários para explicar *por que* algo foi feito de determinado modo.
- Documentar APIs, algoritmos complexos e efeitos colaterais não óbvios.

## Responsabilidade única

- Cada função deve fazer exatamente uma coisa.
- Funções devem ser pequenas e focadas.
- Se uma função precisa de comentário para explicar o que faz, deve ser dividida.

## DRY (Don't Repeat Yourself)

- Extrair código repetido em funções reutilizáveis.
- Compartilhar lógica comum por abstração adequada.
- Manter uma única fonte da verdade.

## Estrutura limpa

- Manter código relacionado junto.
- Organizar em hierarquia lógica.
- Usar convenções consistentes de nomes de arquivos e pastas.

## Encapsulamento

- Ocultar detalhes de implementação.
- Expor interfaces claras.
- Mover condicionais aninhadas para funções bem nomeadas.

## Manutenção da qualidade do código

- Refatorar continuamente.
- Corrigir dívida técnica cedo.
- Deixar o código mais limpo do que estava.

## Testes

- Escrever testes antes de corrigir bugs.
- Manter testes legíveis e sustentáveis.
- Testar casos extremos e condições de erro.

## Controle de versão

- Escrever mensagens de commit claras.
- Fazer commits pequenos, atômicos e focados.
- Usar nomes de branch significativos.

---
> Source: [adriannoes/product-experiment-tracker](https://github.com/adriannoes/product-experiment-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
