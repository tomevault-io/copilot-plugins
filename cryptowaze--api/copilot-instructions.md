## development-rules

> Regras de desenvolvimento do projeto - código, arquitetura e UI


# Regras de Desenvolvimento

## Código

- **Sem comentários**: Código limpo e autoexplicativo. Nomes, funções e estrutura devem transmitir a intenção.
- **Código em inglês**: Variáveis, funções, tipos, nomes de arquivos e mensagens técnicas sempre em inglês.
- **Interface em português**: Textos exibidos ao usuário (labels, mensagens, documentação de produto) em português.
- **Sem emojis**: Manter tom profissional em toda a aplicação.

## Produção e Console

- **Console limpo em produção**: Não usar `console.log`, `console.debug` ou `console.info` em código que roda em produção. Usar logger configurável que em produção não polua o console, ou remover/guardar com flag de ambiente.

## Visual e UI

- **Sem degradês exagerados**: Evitar gradientes chamativos ou desnecessários.
- **Elementos gráficos essenciais**: Incluir apenas elementos que agreguem à usabilidade ou identidade definida. Evitar decoração supérflua.

## Arquitetura e Padrões

- **SOLID**: Respeitar Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion em módulos e serviços.
- **Clean Architecture**: Manter camadas (domain, use cases, adapters, frameworks). Dependências apontando para dentro; regras de negócio independentes de frameworks e UI.
- **Clean Code**: Funções pequenas e focadas, nomes expressivos, baixo acoplamento, alta coesão. Tratar o código como se fosse escrito por Robert C. Martin (Uncle Bob).
- **Não alterar o que não foi pedido**: Modificar apenas arquivos e trechos necessários para a tarefa solicitada. Evitar refatorações ou “melhorias” não solicitadas.

## Documentação

- **Docs explicativas**: Manter documentação que explique decisões, fluxos e uso das APIs quando fizer sentido.
- **Swagger**: Manter especificação OpenAPI/Swagger atualizada para endpoints da API.

---
> Source: [CryptoWaze/API](https://github.com/CryptoWaze/API) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
