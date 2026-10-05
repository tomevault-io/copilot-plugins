## na-lata

> Este é um template de aula, não um app pronto. Todos os alunos devem escrever Flutter e revisar um PR. São livres para criar novas tasks com problema, comportamento esperado, critérios de aceite e plano de testes. Preservar as entregas das outras issues; não resolver o backlog inteiro como efeito colateral.

# Na Lata — práticas para agentes e colaboradores

## Objetivo pedagógico

Este é um template de aula, não um app pronto. Todos os alunos devem escrever Flutter e revisar um PR. São livres para criar novas tasks com problema, comportamento esperado, critérios de aceite e plano de testes. Preservar as entregas das outras issues; não resolver o backlog inteiro como efeito colateral.

## Antes de editar

Ler a issue e `docs/CONTRATOS.md`, `docs/CLIENTES.md` e `docs/DESIGN_SYSTEM.md`. T01–T24 são IDs pedagógicos; números GitHub estão em `docs/ISSUES_GITHUB.md`. Trabalhar na pasta da feature. Alterações em core, clientes, navegação, pubspec e SQL devem ser coordenadas com o professor. Uma solicitação explícita do professor para mudar esses arquivos já autoriza a alteração.

## Arquitetura e dados

- Widget usa repositório por AppScope; nunca acessa Supabase/HTTP diretamente.
- Repositórios Supabase recebem BackendClient. Usar `client.request((db) => ...)` e `client.requireUserId`; não criar clientes globais por feature.
- Open Food Facts usa JsonApiClient injetado; parsing pertence ao repositório.
- Não iniciar chamadas ou escritas em build. Controlar loading e verificar mounted depois de await antes de atualizar widget.
- Respeitar timeout e não repetir gravação automaticamente; reconciliar resultado após timeout.
- Não transformar erro em sucesso/lista vazia. Exibir AppFailure.message; não expor payloads, tokens ou stack traces na UI.
- Validar entradas no app e manter constraints/RLS no banco. Não usar chave administrativa, service_role ou secret no cliente.
- Identidade vem da sessão. Update/delete com zero linhas não é confirmação de sucesso.
- Demo e Supabase são modos explícitos; não trocar para fake silenciosamente.

## Interface

Usar `lib/design_system/na_lata_ui.dart`, tokens e tema compartilhado. Prever loading, vazio, erro e sucesso. Ícones acionáveis têm tooltip; evitar informação só por cor. Conferir 320px e texto 200%. Os componentes são visuais; regras de avaliação continuam nas tarefas.

## Testes e conclusão

Testes fazem parte da funcionalidade: automatizar parsing, validações, agregações e interações relevantes. Usar fakes/MockClient, sem rede ou banco real na suíte unitária. Registrar no PR os cenários manuais (sucesso e erro/vazio), dispositivo e modo. Não criar teste que apenas repete a implementação. Rodar `dart format lib test`, `flutter analyze` e `flutter test`. Mudanças em plugins/Android/bootstrap também pedem build Android.

## Git e revisão

Uma issue por branch, commits focados e PR com Closes #NUMERO. Não incluir configurações locais/segredos. Pedir revisão de colega; professor coordena merges. Respeitar contribuições alheias e não forçar push sem autorização específica.

Para commits feitos em nome do professor nesta máquina/pasta SENAI: **professormatheusq <matheus.quost@edu.sc.senai.br>**. Conferir `git var GIT_AUTHOR_IDENT` e `git var GIT_COMMITTER_IDENT` antes de commitar. Alunos mantêm suas próprias identidades; não atribuir contribuições deles ao professor.

---
> Source: [ADSSTech/na_lata](https://github.com/ADSSTech/na_lata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
