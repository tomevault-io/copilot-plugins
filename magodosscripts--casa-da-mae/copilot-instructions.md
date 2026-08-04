## casa-da-mae

> > Este arquivo é a fonte de verdade para quem pega o projeto agora. A pasta

# Casa da Mãe — guia completo do projeto

> Este arquivo é a fonte de verdade para quem pega o projeto agora. A pasta
> `docs/` (A–O) tem o design original detalhado, mas está **parcialmente
> desatualizada** — quando este arquivo e a `docs/` divergirem, vale este.

## O que é

PWA mobile-first de rotina doméstica para uma família de 8 pessoas, com
etiquetas NFC coladas pela casa e um sistema de **mesada gamificada** para as
crianças. Nome do sistema: **"Casa da Mãe"** (não usar logo; sempre o nome
escrito). Idioma de tudo — UI, código, commits: **português**.

- **Produção**: https://casaos-eight.vercel.app (projeto Vercel `casaos`)
- **Repositório**: https://github.com/Magodosscripts/casa-da-mae (privado)
- **Deploy**: push na `main` = deploy automático em produção (Vercel Git).
  Também dá manual: `npx vercel --prod --yes`.

## Stack

- **Next.js 16** (App Router, Server Components, Server Actions) + TS strict + Tailwind v4
- **Supabase**: Postgres (região `aws-sa-east-1`), Auth (login por **senha**),
  Storage (bucket privado `mesada-fotos`)
- **`pg` Pool direto** no Postgres (pooler porta 6543) para as queries do app —
  o client REST do Supabase só é usado para Auth e Storage
- **Vercel** com `regions: ["gru1"]` (São Paulo — NÃO remover, é o que segura a
  latência; sem isso as funções vão para os EUA e cada tela custa ~2s)
- **OpenAI** (`gpt-4o-mini` com visão) para a prévia de IA das fotos da mesada
- Camadas: `src/domain` (regras puras) → `src/application` (use-cases) →
  `src/infrastructure` (banco, auth, storage) → `src/app` (telas/rotas)

## Os 8 moradores

| Nome | Papel | Observações |
|---|---|---|
| Michella (40) | `house_admin` — "a mãe" | Dona da casa; aprova fotos e ajusta dinheiro |
| Gabriel | `tech_admin` | Autor original (quem te passou o projeto) |
| Henrique | `resident` | |
| Naty (26) | `resident` | Nora |
| **Rafael (17)** | `resident` + mesada | **Sem NFC no celular** → permissão `allowance.app` deixa ele tirar a foto pelo app |
| **Luiz (12)** | criança + mesada | |
| **Carlos (10)** | criança + mesada | |
| **Luiza (8)** | criança + mesada | Tem quarto próprio (separado dos meninos) |

Logins: `nome@casa.com.br` + senha simples (palavra+2 dígitos, definidas por
`scripts/set-passwords.mjs` — as senhas atuais só a família tem; o script gera
novas e imprime UMA vez). Crianças veem uma barra de navegação reduzida
(Mesada + Perfil). `/perfil` só troca senha — o resto foi removido de propósito.

## A mesada (o coração atual do sistema)

Regras pedidas pela família — **não relaxar sem pedido explícito**:

1. **Toda tarefa exige foto NOVA da câmera** (getUserMedia ao vivo em
   `src/app/mesada/camera.tsx`; nunca `<input type=file>`, senão dá para pegar
   da galeria).
2. **Toda tarefa exige a etiqueta NFC** (`requires_tag`): no app aparece só
   "Vá até a etiqueta e encoste o celular", sem botão. Encostar no NFC
   (`/t/[code]`) abre a tela da foto. **Exceção**: Rafael (`allowance.app`)
   tem o botão no app.
3. **Tarefa é única no dia**: quem faz primeiro leva; os outros veem "Fulano
   já fez". Índice único `(household_id, task_key, ref_date, slot)`.
4. **Louça tem 2 turnos reais** (`slots_per_day=2`): manhã/tarde (até 17:59) e
   noite. E é **opcional** (`mandatory=false`) — fazer ganha R$10, não fazer
   não perde nada.
5. As demais são **obrigatórias e diárias**: se NINGUÉM fizer, todos os
   elegíveis perdem o valor (penalidade = mesmo valor da tarefa).
6. **Dois quartos**: `room` = meninos (Luiz, Carlos, Rafael); `room_luiza` =
   só a Luiza. Tabela `allowance_task_participants` — tarefa sem linhas vale
   para todos; com linhas, só para os listados (telas, ações e penalidades
   respeitam isso).
7. **Dinheiro só entra quando a mãe aprova a foto** (`/mesada`, seção
   Aprovar). Depois de aprovar/recusar, **a foto é apagada do storage**
   (pedido: não ocupar espaço). Recusa devolve a vaga da tarefa.
8. **IA**: ao subir a foto, `after()` chama `analise-foto.ts` (OpenAI) e grava
   `ai_ok`/`ai_resumo` no lançamento — a mãe vê "✨ IA: ..." junto da foto. A
   decisão é sempre dela. Sem `OPENAI_API_KEY`, o app funciona sem a prévia.
9. **Meia-noite (cron)**: `/api/cron/fechar-dia` (Vercel Cron 03:00 UTC =
   00:00 SP, autenticado por `CRON_SECRET`): aprova sozinho tudo que a mãe não
   revisou no dia e aplica as penalidades. Não existe botão manual.
10. **Anti-fraude**: segunda foto da mesma tarefa pendente no dia é recusada
    ("já mandou"); corrida pelo mesmo turno é resolvida pelo índice único.
11. Dinheiro **sempre em centavos (int)**, ledger append-only
    (`allowance_entries`), ranking soma só `status='confirmed'`.

Arquivos-chave: `src/domain/mesada/money.ts` (regras puras: turnos, situação,
penalidade) · `src/infrastructure/mesada/mesada-repository.ts` (todas as
leituras/escritas) · `src/app/mesada/` (tela, actions, câmera) ·
`src/app/t/[code]/` (tela da etiqueta; `tag-mesada-foto.tsx` é a da foto).

## As 6 etiquetas NFC físicas (códigos em produção — NÃO regenerar)

| Etiqueta | Mesada | URL |
|---|---|---|
| Lavar louça | R$10 (opcional, 2 turnos) | `/t/7RBDapMef9UosnSCVPrl` |
| Passear com o João (cachorro) | R$5 | `/t/ErdRNP3hbMDikO2fGn9b` |
| Arrumar quarto dos meninos | R$5 | `/t/8XVM7SaeApBFrdLykHn8` |
| Arrumar quarto da Luiza | R$5 | `/t/ErlwvMPyLKY1FdglI58N` |
| Comida + água do João | — (rotina, 1x/dia) | `/t/Mzq3vGydc8xPB5d7GxrZ` |
| Lavagem de roupa | — (rotina) | `/t/NGkRWX3YQJjFhoLQSQnK` |

Prefixo: `https://casaos-eight.vercel.app`. As etiquetas físicas já podem
estar gravadas com esses códigos — mudá-los quebra a casa inteira.

## Permissões

`hasPermission(ctx, key)` — papéis (`role_permissions`) + overrides por pessoa
(`user_permissions`). As da mesada: `allowance.view` (todos os moradores),
`allowance.participate` (as 4 crianças/adolescentes), `allowance.manage` (só a
mãe), `allowance.app` (só o Rafael). `getAuthContext`
(`src/infrastructure/auth/context.ts`) resolve tudo numa query só, com React
`cache()` por request, e roda a validação do JWT **em paralelo** com a query —
não serializar de novo.

## Performance (conquistada a muito custo — não regredir)

- `regions: ["gru1"]` no `vercel.json`
- Middleware (`src/proxy.ts`) **não vai à rede** por request: valida o cookie
  localmente e só chama o Supabase quando o token está a <5 min de expirar
- Pool `max: 5` (max:1 serializava `Promise.all`)
- Queries de página sempre em `Promise.all`
- `experimental.staleTimes.dynamic: 30` → troca de abas ~50ms
- Sem `backdrop-blur` em cards (só na barra e no leque)
- Números de referência (medidos): primeira carga 120–240ms; abas ~50ms;
  cold start do plano gratuito ~1s no primeiro acesso após horas parado

## Banco

Migrations `supabase/migrations/0001..0014` aplicadas por runner próprio
(`npm run db:migrate`; nunca editar migration aplicada — criar nova). Seed
idempotente (`npm run db:seed`) substitui `{{EMAIL_*}}` pelos `SEED_EMAIL_*`
do `.env.local`. Pontos de atenção:

- `events` tem trigger anti-delete (`events_no_delete`) — para limpar, siga o
  padrão de `scripts/ir-para-producao.mjs` (disable/enable na transação)
- `nfc_tags.public_code` tem check de tamanho ≥16
- Fuso de tudo: `America/Sao_Paulo` (coluna `households.timezone`)
- `ref_date` dos lançamentos = data no fuso da casa, não UTC

## Testes e simulações

- `npm run typecheck` · `npm run lint` · `npm run build` — manter limpos
- `SIMULACAO=1 npx vitest run tests/simulacao-mesada.test.ts` — simula TODAS
  as regras da mesada contra o banco real (17 checagens; **muta o banco** —
  só guarda `SIMULACAO=1` a protege do `npm test`)
- `node scripts/sim-telas.mjs <pasta>` — entra como cada morador via magic
  link (sem tocar nas senhas) e confere o que cada tela mostra (23 checagens);
  precisa do dev server local. Atenção: `innerText` vem com o `uppercase` do
  CSS aplicado — compare em minúsculas
- `node scripts/sim-veloc.mjs` — mede a velocidade real em produção
- Playwright em headless: câmera fake exige
  `--use-fake-ui-for-media-stream --use-fake-device-for-media-stream` e mesmo
  assim é instável — teste câmera em aparelho real

## Armadilhas conhecidas (todas já mordidas uma vez)

1. **Dev server órfão na porta 3000** servindo 500 velho — mate o processo
   antes de subir outro (`Get-NetTCPConnection -LocalPort 3000`)
2. **`Permissions-Policy: camera=()` bloqueia a câmera do próprio app** — tem
   que ser `camera=(self)` (`next.config.ts`)
3. `backdrop-filter` cria containing block: overlay `fixed` dentro de card com
   blur fica preso — a câmera usa `createPortal(document.body)` por isso
4. CSP restritiva: nada de CDN/script externo; a prévia de IA roda no servidor
5. `.env.local` **jamais** no git (conferir `git status` antes de commit)

## Variáveis de ambiente (`.env.example` documenta; valores no Vercel + `.env.local`)

> **Pegando o projeto agora?** O `.env.local` completo (com as chaves reais)
> está num **gist secreto** — peça o link ao Gabriel. Baixe o conteúdo para
> `.env.local` na raiz e `npm run dev` já funciona. O link não está neste
> repositório de propósito: o repo é público e as chaves seriam revogadas
> automaticamente pelos provedores se aparecessem aqui.

`DATABASE_URL` (pooler 6543) · `NEXT_PUBLIC_SUPABASE_URL` ·
`NEXT_PUBLIC_SUPABASE_ANON_KEY` · `SUPABASE_SERVICE_ROLE_KEY` (server-only) ·
`OPENAI_API_KEY` (IA das fotos) · `CRON_SECRET` (cron da meia-noite) ·
`SEED_EMAIL_*` (e-mails reais dos 8) · `N8N_WEBHOOK_SECRET`.

⚠️ **Segurança — fazer cedo**: a `SUPABASE_SERVICE_ROLE_KEY` e a
`OPENAI_API_KEY` atuais já circularam em chats durante o desenvolvimento.
Rotacionar as duas (dashboard do Supabase → API keys; platform.openai.com) e
atualizar `.env.local` + Vercel (`npx vercel env`) é a primeira tarefa boa de
manutenção.

## O que existe além da mesada

Dashboard (`/`), tarefas da casa (`/tarefas`, hoje no leque da bola central),
processos de lavanderia (internos — fora do menu de propósito), cuidados do
cachorro João com janelas/deadline, histórico auditável com correção
versionada (original nunca é apagado), admin de etiquetas com QR
(`/admin/ntags`), simulador (`/admin/simulador`), notificações, PWA
instalável, integração Alexa (consulta, opcional) e webhooks n8n (automação
temporal; o n8n não está hospedado — os webhooks têm HMAC e estão prontos).

## Convenções

- Commits em português, terminados com
  `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>` (ajuste para o
  modelo que estiver em uso)
- Código comentado em português, no tom "explica o porquê, não o quê"
- Design: neutro com acento violeta (`--acento`), cor forte só para estado
  (atrasado/agora/feito); estado nunca comunicado só por cor
- Mobile-first sempre (a família usa celular; iPhone 13 é o viewport de teste)

---
> Source: [Magodosscripts/casa-da-mae](https://github.com/Magodosscripts/casa-da-mae) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
