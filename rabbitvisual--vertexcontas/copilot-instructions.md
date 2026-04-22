## vertexcontas

> Regra GFP (Gestão Financeira Pessoal): terminologia, pilares e tom para todo o produto Vertex Contas. Aplicar em UI, copy, tours e documentação.

# GFP - Gestão Financeira Pessoal (Vertex Contas)

Toda interface, copy e documentação do produto devem seguir os princípios e a terminologia abaixo para um GFP profissional e acessível a usuários com pouco conhecimento em finanças.

## Terminologia obrigatória

| Termo | Significado | Uso |
|-------|-------------|-----|
| **Capacidade mensal** | O que sobra para gastar após receitas e despesas fixas (Minha Renda). No Pro: já descontando o comprometido com metas. | Dashboard, Minha Renda, tours, tooltips. |
| **Minha Renda** | Planejamento mensal: receitas esperadas e despesas fixas. Não é extrato; é a base do cálculo da capacidade. | Menu, títulos, breadcrumb. |
| **Extrato** | Transações reais nas contas (entradas e saídas). Registro do que de fato entrou e saiu. | Menu, listagem de transações. |
| **Metas** | Objetivos de economia com valor alvo e progresso rastreável. Podem ter contribuição automática e vínculo com despesas. | Menu, formulários, notificações. |
| **Saldo total** / **Saldo das contas** | Soma do saldo de todas as contas do usuário. | Dashboard (Free e Pro). Evitar "Estoque (Saldo)". |
| **Receitas do mês** | Total de receitas (planejado em Minha Renda ou realizado no período, conforme contexto). | Cards do dashboard. |
| **Despesas do mês** | Total de despesas no período. | Cards do dashboard. |
| **Reserva de emergência** | Saldo equivalente a X meses de despesas; conceito GFP para segurança. | Gamification, relatórios, textos didáticos. |

## Pilares de copy (GFP)

- **Registro e categorização:** incentive o registro diário e a categorização dos gastos para entender para onde vai o dinheiro.
- **Fluxo de caixa:** visão clara de saldo, receitas e despesas; projeção quando aplicável.
- **Metas e orçamentos:** metas de poupança com progresso; orçamentos por categoria com alertas.
- **Regra 50/30/20:** quando relevante, cite: 50% necessidades, 30% desejos, 20% poupança e metas.
- **Reserva de emergência:** foque no saldo positivo para construir reserva e segurança.

## Tom e usabilidade

- **Idioma:** português (Brasil) em toda a UI.
- **Clareza:** frases curtas; evite jargão desnecessário.
- **Didática:** onde ajudar, use blocos "Como usar", "Dica" ou tooltips explicando conceitos (ex.: capacidade = receitas − despesas fixas; planejamento vs extrato).
- **Consistência:** Free e Pro devem usar a mesma nomenclatura nos blocos equivalentes (Saldo total, Receitas do mês, Despesas do mês, Capacidade mensal). Pro pode acrescentar "Comprometido com metas", "Destinado a metas este mês", "Balanço mensal".

## Referências

- `GFP.md`: melhores práticas (registro diário, categorização, 50/30/20, projeção, reserva).
- `Modules/Core/resources/views/goals/readme.md`: princípios técnicos de metas (fonte única de verdade, capacidade realista, metas concluídas, rastreabilidade).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RabbitVisual) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
