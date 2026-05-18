## flowdesks

> > Esse arquivo é a fonte de verdade para o agente de IA entender o projeto, manter o padrão e evitar regredir decisões já tomadas. **Sempre leia antes de editar código.**

# FlowDesks — Contexto do Projeto

> Esse arquivo é a fonte de verdade para o agente de IA entender o projeto, manter o padrão e evitar regredir decisões já tomadas. **Sempre leia antes de editar código.**

---

## 1. Visão geral

**FlowDesks** é um PWA fullstack para gestão operacional de equipes em campo: calendário, alocações, controle de ponto com geofence, evidências fotográficas, ajustes de ponto e exportação para pagamento.

- **Frontend:** Angular 17 standalone + Signals + Reactive Forms + Angular Material + FullCalendar + Tailwind
- **PWA:** `@angular/service-worker` + `ngsw-config.json`
- **Backend:** Supabase (Auth + Postgres + RLS + RPC + Edge Functions + Storage)
- **Timezone:** `America/Fortaleza` no frontend, `timestamptz` em UTC no banco
- **Idiomas:** pt-BR, en, es (default atual: en) via `TranslatePipe` (`| t`)
- **Identificador da app no Supabase:** `profiles.app_code = 'FLOWDESKS'` — usuários de outras apps são bloqueados no login
- **Isolamento físico (migration 030):** TODAS as tabelas operacionais (`assignments`, `employees`, `locations`, `activity_types`, `assignment_attendances`, `assignment_attendance_requests`, `assignment_work_photos`, `notifications`, `reassignment_logs`) têm `app_code text not null default 'FLOWDESKS' check (app_code = 'FLOWDESKS')` + trigger BEFORE INSERT que força o valor. Helper `public.is_flowdesks_user()` é usado em toda RLS. Rollback em `supabase/sql/030_flowdesks_lockdown_rollback.sql`

---

## 2. Perfis e roteamento

Definidos em `src/app/shared/models/role.model.ts`:

| Role          | Home          | Shell                                              |
| ------------- | ------------- | -------------------------------------------------- |
| `SUPER_ADMIN` | `/director`   | `features/director/director.shell.ts`              |
| `ADMIN`       | `/admin`      | `features/admin/admin.shell.ts`                    |
| `COLLABORATOR`| `/me/ponto`   | `features/collaborator/collaborator.shell.ts`      |

Guards: `core/guards/auth.guard.ts` e `core/guards/role.guard.ts`. Rotas em `src/app/app.routes.ts`.

---

## 3. Estrutura de diretórios

```
src/app/
  app.component.{ts,html,scss}     # toolbar global, banner offline, idioma, logout
  app.config.ts                    # providers (router, animations, sw, snackbar)
  app.routes.ts                    # rotas + guards por role
  core/
    guards/                        # authGuard, roleGuard
    i18n/i18n.service.ts           # dicionário pt-BR/en/es + signal de idioma
    supabase/                      # client, session.store (signals), services e RPCs
    ui/                            # toast.service, pwa-notification.service
  shared/
    components/                    # confirm-dialog, loading
    models/                        # assignment.model.ts, role.model.ts
    pipes/                         # t.pipe (i18n), tz-date.pipe (Fortaleza)
  features/
    auth/login.page.ts
    admin/
      admin.shell.ts               # nav superior do ADMIN
      calendar/                    # FullCalendar + diálogos (assignment, reassign, payment-summary)
      employees/                   # CRUD colaboradores
      catalogs/                    # locations.page, activity-types.page
      work-photos/                 # galeria de fotos com filtros + ZIP
    collaborator/
      collaborator.shell.ts        # apenas <router-outlet/>
      my-calendar.page.ts          # "ponto" (check-in/out + geofence)
      requests.page.ts             # solicitações de ajuste
      profile.page.ts
      assignment-detail.dialog.ts
    director/
      director.shell.ts
      admins.page.ts

supabase/
  migrations/                      # SQL versionado (já em 029_*)
  functions/create-collaborator/   # Edge Function (Service Role) p/ criar usuário no Auth
  sql/                             # scripts auxiliares (make_admin, seed, ...)
```

---

## 4. Padrões obrigatórios

### Angular
- **Sempre standalone**, com `imports: [...]` no decorator. Nunca criar NgModule.
- **Sempre `ChangeDetectionStrategy.OnPush`** em componentes novos.
- **Estado em Signals** (`signal`, `computed`, `effect`). Evitar `BehaviorSubject` se não houver razão forte.
- **Reactive Forms tipados** via `formBuilder.nonNullable.group(...)`.
- **Templates** preferem control flow novo (`@if`, `@for`, `@switch`). Não usar `*ngIf`/`*ngFor` em código novo.
- **Lazy load por rota** com `loadComponent: () => import(...).then(m => m.X)`.
- **i18n:** todo texto visível usa `{{ 'chave' | t }}`. Adicionar a mesma chave nos 3 idiomas em `i18n.service.ts`.
- **Datas no UI:** sempre via `tzDate` pipe ou formatadas com `Intl.DateTimeFormat` em `America/Fortaleza`.
- **Naming:**
  - serviços `*.service.ts`, dialogs `*.dialog.ts`, páginas `*.page.ts`, shells `*.shell.ts`, models `*.model.ts`
  - seletores com prefixo `app-` (configurado em `angular.json`)

### Estilos
- **Material 17** com tema custom (cyan + teal) em `src/styles.scss`.
- **Tailwind** habilitado (`tailwind.config.js`) com paleta `brand.50..900` e shadow `soft`. Usar Tailwind para utilitários; Material para componentes.
- **Tokens CSS globais** em `:root`:
  - `--bg-a #f6fbff` / `--bg-b #edf8ff` / `--card #fff` / `--stroke #dbe7f0` / `--text #1f2937` / `--muted #6b7280`
- **Classe utilitária `surface-card`** = card com borda `--stroke`, raio 14px e shadow soft. Reutilizar em vez de inventar variações.
- **Dialogs** com `panelClass`:
  - `assignment-dialog-panel` (alocação)
  - `reassign-dialog-panel` (remanejamento)
  - `confirm-dialog-panel` (confirmação)
  - Estilos já configurados em `styles.scss` (sticky actions, safe-area-inset-bottom etc.). Usar essas classes ao criar diálogos novos.

### Supabase
- Cliente único em `core/supabase/supabase.client.ts` usando `sessionStorage` (não `localStorage`) para não vazar sessão entre abas/usuários compartilhados.
- Acesso a dados **sempre via service** em `core/supabase/*.service.ts` — páginas não chamam `supabase.from(...)` diretamente.
- **RLS é a fonte de autorização.** Frontend filtra por UX; nunca confiar no client para segurança.
- **RPCs:** `reassign_assignment`, attendance/geofence punch, etc. — sempre que houver regra transacional, criar RPC em vez de orquestrar no client.
- **Tratamento de erro de overlap:** quando vier `assignments_no_overlap`, mostrar mensagem amigável (já existe em `assignments.service.ts`).
- **Edge Function `create-collaborator`** é o único caminho seguro para criar usuário (usa Service Role). Nunca expor Service Role no client.
- **Migrations** são append-only e numeradas (`NNN_descricao.sql`). Próxima livre = `030_*`. Nunca editar migration já aplicada — criar nova.

---

## 5. Domínio (modelos chave)

`src/app/shared/models/assignment.model.ts`:

- `Profile` — usuário (role, app_code, active)
- `Employee` — dados operacionais (matrícula, cargo, telefone)
- `Location` — local com geofence (lat/lng + raio em metros + maps_url)
- `ActivityType` — tipo de serviço
- `Assignment` — alocação com janela `start_at..end_at`, valores (`hourly_rate`, `daily_rate`, `fixed_wage`, `expenses`, `extras`, `deductions`, `total_amount`), status `PLANNED|CONFIRMED|CANCELLED`, `recurrence_group_id`, e relacionamentos populados (`employee`, `location`, `activity_type`, `attendance`, `work_photos`)
- `AssignmentAttendance` — ponto: `check_in_at`, `check_out_at`, status `NOT_STARTED|CHECKED_IN|DONE`, fotos antes/depois
- `AssignmentWorkPhoto` — foto com phase `BEFORE|AFTER`, GPS (lat/lng/accuracy/heading), local capturado
- `AttendanceAdjustmentRequest` — solicitação de ajuste de ponto (`PENDING|APPROVED|REJECTED`)
- Cargos válidos do operacional: `REGULAR SERVICES`, `EXTRA SERVICES`, `EXTRA SERVICES - PROJECT GUYS`

---

## 6. Funcionalidades já entregues (não regredir)

1. Login email/senha, persistência por aba, bloqueio por `app_code`, redirect por role
2. Director: lista de admins
3. Admin:
   - CRUD de colaboradores (com Edge Function p/ criação) e desativação lógica
   - CRUD de locais (parsing de coordenadas a partir do link Maps + geofence)
   - CRUD de tipos de atividade (dedup automático)
   - Calendário operacional (FullCalendar) com filtros, drag&drop de datas, criação/edição/remanejamento/exclusão (incl. série de recorrência), separação visual entre passado/hoje/futuro
   - Galeria de fotos com filtros (período, colaborador, local, fase, GPS) e download em ZIP
   - Aprovação/rejeição de solicitações de ajuste
   - Modal de resumo de pagamento com totais por colaborador e geral, exportação operacional + planilha
4. Collaborator:
   - "Ponto" (agenda + check-in/check-out com validação de geofence e precisão GPS)
   - Solicitações de ajuste
   - Upload de fotos (antes/depois) com GPS e exclusão pelo próprio colaborador
   - Perfil
5. Notificações (PWA) — colaborador recebe mudança de agenda; admin recebe conclusão de trabalho
6. Banner de modo offline + cache da última agenda em IndexedDB (`idb-keyval`)
7. Anti-conflito de horário no banco (`assignments_no_overlap`)

---

## 7. Convenções de UI / UX

- **Toolbar global** (`app.component.html`) é sticky, com `surface-card` translúcido + blur. Mantém menu de idioma e conta.
- **Shells por role** controlam só a navegação interna; a toolbar global é compartilhada.
- **Mobile-first responsivo**: media queries em `app.component.scss` e `styles.scss` cobrem `< 760px` e `< 900px`. FullCalendar tem ajustes específicos para mobile já no SCSS global.
- **Idioma e conta** ficam em `mat-menu` triggerado por botão stroked.
- **Toasts** via `ToastService` com classes `toast-success|error|info` (cores já definidas).
- **Confirmação destrutiva** sempre via `ConfirmDialog` em `shared/components/confirm-dialog`.

---

## 8. Decisões e constraints históricas

- **Dois nomes possíveis para anon key**: o cliente lê tanto `environment.supabaseAnon` quanto `environment.supabaseAnonKey`. Manter compatibilidade ao mexer.
- **`sessionStorage`** é proposital — _não trocar para `localStorage`_ sem entender o impacto em compartilhamento de sessão.
- **Default de idioma** no app está como `en` (decisão de produto, ainda que o app seja brasileiro).
- **Cargo** é uma das 3 categorias acima — não introduzir variações livres sem alinhar.
- **Saída de ponto sem limite de horário máximo** é intencional (migration 027).
- **Notificações restritas a calendário** (migrations 016/017) — notificações legadas foram removidas, não ressuscitar.

---

## 9. Comandos do dia a dia

```bash
npm run start          # ng serve em http://localhost:4200
npm run build          # build produção
npm run test           # karma (raramente usado)

# Supabase
supabase db push                                # aplica migrations
supabase functions deploy create-collaborator   # publica Edge Function
```

Variáveis em `src/environments/environment{,.prod}.ts`:
- `supabaseUrl`, `supabaseAnonKey` (ou `supabaseAnon`), `timezone`

---

## 10. Roadmap (referência: `docs/proposta-flowdesks.md`)

- **Fase 2 (consolidação):** dashboards, relatórios gerenciais, aprovação em lote de ajustes, exportação financeira por centro de custo
- **Fase 3 (escala):** BI, integração folha, integração WhatsApp/email, módulo cliente final

Ao adicionar feature nova, perguntar de qual fase ela faz parte e verificar se já não há trabalho relacionado começado.

---

## 11. Antes de fazer qualquer mudança

1. Ler este arquivo
2. Reusar componente/serviço existente antes de criar novo
3. Adicionar chaves i18n nos 3 idiomas
4. Manter `OnPush` + Signals
5. Para mudanças de banco: nova migration `NNN_descricao.sql`, nunca editar antiga
6. Não expor Service Role no client; criar Edge Function se precisar de privilégio
7. Validar mobile (≤ 760px) — operação acontece em campo

---
> Source: [Joot4/flowdesks](https://github.com/Joot4/flowdesks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
