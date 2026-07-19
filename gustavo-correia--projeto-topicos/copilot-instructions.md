## projeto-topicos

> Este repositório contém o app **Assinaturas Ninja**, um aplicativo Flutter offline de gestão financeira focado em assinaturas recorrentes.

# AGENTS.md

## Projeto

Este repositório contém o app **Assinaturas Ninja**, um aplicativo Flutter offline de gestão financeira focado em assinaturas recorrentes.

O objetivo é entregar um app acadêmico funcional, apresentável e fácil de explicar, mantendo CRUD, dashboard, relatórios simples, onboarding e persistência local.

## Papel do Agente

Atue como desenvolvedor Flutter responsável por:

- Implementar funcionalidades dentro do escopo.
- Manter a arquitetura simples e o código legível.
- Corrigir erros sem introduzir complexidade desnecessária.
- Garantir que o app continue executável.
- Atualizar testes e documentação quando o comportamento mudar.

## Leitura Obrigatória

Antes de alterar o app, consulte:

- `docs/README.md`
- `docs/PROJECT_BRIEF.md`
- `docs/UI_GUIDE.md`
- `docs/ARCHITECTURE.md`
- `docs/TASKS.md`

## Regras Gerais

1. Não adicione backend, Firebase, login ou sincronização na nuvem.
2. Não implemente integração bancária, pagamentos reais ou notificações complexas.
3. Mantenha o app offline.
4. Prefira código simples a abstrações excessivas.
5. Use nomes claros em inglês no código.
6. Use português brasileiro nos textos visíveis e na documentação.
7. Preserve a identidade escura e a marca existente.
8. Não insira dados demonstrativos automaticamente no primeiro uso.

## Stack

- Flutter / Dart
- Provider para estado
- SharedPreferences para persistência local
- intl para formatação
- uuid para identificadores

## Estrutura

```txt
lib/
  main.dart
  models/
  providers/
  screens/
  services/
  utils/
  widgets/
```

## Funcionalidades Atuais

- Onboarding com início vazio ou demonstração opcional.
- Dashboard de assinaturas ativas.
- CRUD, detalhes e mudança rápida de status.
- Busca, filtros e ordenação.
- Relatórios por categoria e insights.
- Configurações locais e gerenciamento de dados.
- Identidade visual própria e ícone Android.

## Regras de Negócio

- Apenas assinaturas ativas entram em totais, contagem ativa, categoria mais cara e economia potencial.
- Pausadas e canceladas ficam armazenadas, mas não entram nos totais ativos.
- Vencimento próximo significa cobrança em até 5 dias.
- Vencimento hoje deve receber destaque forte.
- Valor deve ser maior que zero.
- Dia de vencimento deve ficar entre 1 e 31.
- Dados de demonstração dependem de uma ação explícita do usuário.

## Critérios de Aceite

Antes de considerar uma mudança pronta, execute as verificações apropriadas:

```bash
flutter analyze
flutter test
flutter build apk --debug
```

Também confirme, quando a alteração atingir a experiência:

- Fluxo de onboarding.
- Fluxos de cadastro, edição e exclusão.
- Cálculos de dashboard e relatórios.
- Persistência local após reinício.
- Legibilidade em tela de celular.

---
> Source: [Gustavo-Correia/projeto-topicos](https://github.com/Gustavo-Correia/projeto-topicos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
