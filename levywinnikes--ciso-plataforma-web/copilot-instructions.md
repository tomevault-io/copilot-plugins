## ciso-plataforma-web

> Objetivo: garantir que qualquer agente de IA leia documentacao antes de implementar mudancas, e questione discrepancias antes de codar.

# Instrucoes para IA neste repositorio

Objetivo: garantir que qualquer agente de IA leia documentacao antes de implementar mudancas, e questione discrepancias antes de codar.

## Regra de ouro (obrigatoria)

Antes de criar ou alterar codigo, a IA deve:

1. Ler o indice em docs/ai/doc-index.md.
2. Abrir os documentos mapeados para a feature solicitada.
3. Confirmar restricoes tecnicas e de negocio encontradas.
4. **Se encontrar discrepancia entre codigo e documentacao**, PARAR e perguntar ao usuario qual a fonte de verdade antes de prosseguir.
5. So entao propor e aplicar alteracoes.

Se a IA nao encontrar documentacao suficiente, deve primeiro propor a atualizacao da documentacao, obter aprovacao do usuario, e apenas entao implementar.

## Politica de questionamento (obrigatoria)

A IA NUNCA deve "silenciosamente" aceitar inconsistencias. Em qualquer um destes cenarios, ela deve interromper o trabalho e perguntar ao usuario:

- Codigo existente contradiz a documentacao oficial em docs/ai/.
- Pedido do usuario viola uma regra documentada (ex: hardcode de texto, endpoint sem auth, role sem mapeamento).
- Surge um padrao novo nao previsto na documentacao.
- Endpoint API sem autenticacao detectado durante leitura.
- String de UI hardcoded encontrada durante leitura.
- Erro de API retornando texto em portugues hardcoded (deve ser chave `errors.*`).

Ao questionar, sempre apresentar:

1. O que foi encontrado (codigo e/ou documento).
2. Por que parece divergente.
3. Opcoes possiveis (corrigir codigo, atualizar documento, ou aceitar exception com justificativa).

## Fluxo padrao de execucao

1. Entender o pedido e classificar o tipo de mudanca:

- UI/UX
- Formulario/validacao
- Internacionalizacao
- API/servico
- Dados/Prisma
- Testes
- Infra/qualidade

2. Consultar documentacao obrigatoria:

- docs/ai/doc-index.md
- README.md

3. Consultar documentacao especifica por dominio (via indice).

### ⚠️ REGRA ESPECIAL: Mudancas de Autenticacao, Autorizacao e Roles

Se a mudanca envolve qualquer uma das palavras-chave abaixo, LEIA OBRIGATORIAMENTE:

- **docs/ai/access-and-permissions.md** (roles, vinculacao organizacional, matriz de permissoes)
- **docs/ai/security-checklist.md** (padrao obrigatorio para handlers API)
- **src/lib/api-auth.ts** (helpers — NUNCA reinventar)

Palavras-chave que acionam esta regra:

- Role, permission, acesso, autorizacao, authentication, JWT, middleware de rota
- Qualquer mudanca no arquivo `src/features/auth/`
- Criacao de nova rota em `/admin`, `/medico`, `/profissional`, `/clinica`
- Criacao ou alteracao de qualquer arquivo em `src/app/api/**`
- Qualquer filtro `where: { organizationId }`

Se a documentacao nao cobrir o caso, PERGUNTE ao usuario antes de implementar.

### ⚠️ REGRA ESPECIAL: Mudancas em rotas API

Toda rota em `src/app/api/**` exige:

1. Leitura de **docs/ai/security-checklist.md** antes de tocar no arquivo.
2. Primeira linha do handler chama `requireSession()` / `requireAdministrativo()` / `canManageOrg()` / `canManageUser()` de `src/lib/api-auth.ts`.
3. Todas as respostas de erro usam `apiError("errors.<chave>", status)` — chave registrada em ambos JSONs de i18n.
4. Toda entrada externa validada com Zod `safeParse`.
5. Endpoint documentado em `docs/ai/api-routes.md` apos a mudanca.

### ⚠️ REGRA ESPECIAL: Strings de UI

- **Nenhuma string de UI hardcoded.** Sempre `useTranslations` + chave em `src/i18n/messages/{pt-BR,en-US}.json`.
- Mensagens de erro de Zod devem ser chaves (ex: `"errors.required"`), traduzidas em runtime via `useFormError` em `src/i18n/use-form-error.ts`.
- Mensagens de erro de API devem ser codigos `errors.*` retornados como JSON, traduzidos pelo cliente.
- Ao adicionar nova chave, atualizar **ambos** `pt-BR.json` E `en-US.json`.

4. Implementar mantendo os padroes:

- Sem hardcode de textos de interface: usar chaves de traducao.
- Preferir componentes reutilizaveis para comportamentos recorrentes.
- Validacao de entrada via schema quando aplicavel.
- Nao quebrar contratos existentes de tipos e dados.

5. Validar antes de finalizar:

- npm run lint
- npm run build
- npm run test (quando houver impacto funcional)

6. Se algo novo foi decidido, registrar no local correto do indice.

## Politica de atualizacao de documentacao

Sempre que a IA:

- Criar um novo padrao arquitetural ou regra recorrente — registrar em `docs/ai/patterns.md`.
- Criar/alterar endpoint API — atualizar `docs/ai/api-routes.md`.
- Criar/alterar regra de permissao — atualizar `docs/ai/access-and-permissions.md`.
- Criar nova tabela/relacao no Prisma — atualizar `docs/ai/database-structure.md`.
- Criar novo documento — registrar em `docs/ai/doc-index.md`.

Antes de aplicar essas atualizacoes, **pedir confirmacao do usuario** com um resumo da mudanca proposta.

## Criterios de qualidade para PR/entrega

- Mudanca pequena, focada e consistente com o dominio.
- Arquivos novos com responsabilidade clara.
- Sem duplicacao de regra de negocio em multiplos pontos.
- Sem regressao de i18n e sem strings de UI hardcoded.
- Endpoints sempre passam pelo `api-auth.ts`.
- Evidencia de validacao local (lint/build/test).

## Quando criar novo documento

Criar documentacao nova apenas quando:

- Existe uma regra recorrente sem lugar claro no repositorio.
- A feature introduz um novo fluxo arquitetural.
- O custo de manutencao sem documento seria alto.

Sempre atualizar docs/ai/doc-index.md ao criar, mover ou remover documentos.

---
> Source: [levywinnikes/ciso-plataforma-web](https://github.com/levywinnikes/ciso-plataforma-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
