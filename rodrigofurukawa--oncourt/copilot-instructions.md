## oncourt

> Contexto persistente do projeto para agentes de IA (Claude Code, ChatGPT, etc.).

# AGENTS.md — OnCourt

Contexto persistente do projeto para agentes de IA (Claude Code, ChatGPT, etc.).
Mantenha este arquivo atualizado quando decisões de arquitetura ou estado do projeto mudarem.

---

## 1. O que é

**OnCourt** — aplicação web de locação/reserva de quadras esportivas (futsal, tênis,
vôlei, beach tennis) para clubes em São Paulo. Usuário busca clubes, vê quadras e
horários, reserva slots; admin gerencia disponibilidade.

- **Framework:** Django 6.0 (server-side rendering, templates Django).
- **App principal:** `booking`.
- **Projeto/config:** `courton`.
- **Idioma da UI:** Português (BR).

## 2. Stack e decisões de arquitetura

| Camada | Hoje | Protótipo (nuvem) | Produção (alvo) |
|---|---|---|---|
| App Django | local (`runserver`) | **Render** (gunicorn) | **AWS** — App Runner/ECS preferível a EC2 |
| Banco | SQLite (só auth/sessions) | **Supabase Postgres** | **AWS RDS Postgres** |
| Estáticos | `runserver` | WhiteNoise | WhiteNoise ou S3+CloudFront |

**Decisões tomadas (2026-06-29):**
1. **Postgres em todos os ambientes** (paridade dev/prod). Dev local via Docker; nuvem
   troca apenas a `DATABASE_URL`. Nada de saltar SQLite → Supabase → RDS com engines
   diferentes.
2. **Host de protótipo = Render + Supabase** (não Vercel). Vercel é serverless/frontend
   e hostiliza Django (cold start, sem worker, estáticos/admin problemáticos). Render
   trata Django como cidadão de primeira classe.
3. **Supabase é usado só como Postgres gerenciado.** Auth/RLS do Supabase NÃO são usados
   — o Django tem auth próprio (`django.contrib.auth`).
4. **Produção AWS:** RDS sempre. EC2 só se for necessário controle total de SO; caso
   contrário App Runner/ECS Fargate (menos ops).
5. **Criar models Django reais + seed** a partir de `mock_data.py` (ver §4).

## 3. Estado atual (importante)

- ✅ **MVP rodando no banco.** Views e templates usam o ORM; `mock_data.py` foi **removido**.
- ✅ **Schema relacional** em [`booking/models.py`](booking/models.py) — ver §3.1.
- ✅ **Disponibilidade por dia da semana** em [`booking/availability.py`](booking/availability.py):
  os horários são GERADOS por data a partir das janelas (`CourtSchedule`) + `Court.slot_minutes`,
  e o status (available/booked/blocked) considera `Reservation` e `SlotBlock` daquela data.
- ✅ **Reservas persistidas** no model `Reservation` (com unicidade court+date+start_time).
- ✅ **Seed** sem fixtures externas: `python manage.py seed_data [--flush]` →
  3 clubes, 6 quadras, 42 janelas de horário.
- ✅ **`settings.py` env-izado** (`SECRET_KEY`/`DEBUG`/`ALLOWED_HOSTS`/`DATABASE_URL` via
  `.env`), WhiteNoise, `STATIC_ROOT`, flags de segurança fora de DEBUG. Models no admin.
- ⚠️ **Banco atual ainda é SQLite** (fallback): Docker baixado mas pendente de reiniciar o PC.
  Trocar é só definir `DATABASE_URL` (Postgres local ou Supabase) e rodar `migrate` + `seed_data`.

### 3.1 Modelo de dados

- **`Club`** — clube. Local estruturado (`address`, `neighborhood`, `city`, `state`,
  `postal_code`, `latitude`/`longitude`), contato (`phone`, `email`, `website`), `rating`,
  `featured`, `slug`. Propriedades: `.location` (resumo legível) e `.sports` (derivado).
- **`Court`** — quadra (FK `club`). Especificidades: `sport`, `size`, `surface`,
  `is_indoor`, `price_per_hour`, `rating`, `tags`, `slot_minutes` (duração do horário).
- **`CourtSchedule`** — janela de funcionamento por `weekday` (`opening_time`/`closing_time`).
  Pode haver várias por dia (manhã/noite). É a base da geração de horários.
- **`Reservation`** — `user`, `court`, `date`, `start_time`, `status`. Único por
  (court, date, start_time). Propriedades de conveniência p/ templates.
- **`CourtBlock`** — bloqueio da quadra inteira (motivo/prazo). `SlotBlock` — bloqueio de um
  horário específico em uma data.

## 4. Roadmap de migração (status)

1. ✅ **`requirements.txt`** — instalado no venv.
2. ✅ **Env-izar `settings.py`**.
3. ⏸️ **Postgres local** — `docker-compose.yml` pronto; falta reiniciar p/ Docker subir,
   OU apontar `DATABASE_URL` direto pro Supabase.
4. ✅ **Schema relacional** — Club, Court, CourtSchedule, Reservation, CourtBlock, SlotBlock.
5. ✅ **Seed** próprio (`seed_data`), sem `mock_data`.
6. ✅ **Views/templates no ORM** + reservas persistidas + dimensão de data nas telas.
7. ✅ **Deploy Render + Supabase** — no ar em **https://oncourt-6zbu.onrender.com**
   (Render free, web service manual; banco = Supabase Session pooler, us-west-2).
   Passo a passo e detalhes em [`DEPLOY.md`](DEPLOY.md).
8. ⬜ **Produção AWS** — RDS + App Runner/ECS; mesma codebase, env vars diferentes.

> Princípio: MVP → reforços. Não "frameworkizar" cedo. Mudanças mínimas por etapa.

## 5. Layout do projeto

```
OnCourt/
├── manage.py
├── manage.py
├── requirements.txt      # deps (instaladas no venv)
├── docker-compose.yml    # Postgres local (paridade dev/prod)
├── .env / .env.example   # config por ambiente (.env é gitignored)
├── courton/              # config do projeto
│   ├── settings.py       # env-izado (SECRET_KEY/DEBUG/DATABASE_URL/WhiteNoise)
│   ├── urls.py
│   └── wsgi.py / asgi.py
├── booking/              # app principal
│   ├── models.py         # Club, Court, CourtSchedule, Reservation, CourtBlock, SlotBlock
│   ├── availability.py   # geração de horários por data + status (lógica central)
│   ├── admin.py          # models registrados
│   ├── views.py          # usa ORM + availability + dimensão de data
│   ├── urls.py           # rotas em PT: /clubes, /quadras/<id>/reservar, etc.
│   ├── forms.py          # ClubFilterForm, SignUpForm
│   ├── management/commands/seed_data.py   # popula o banco (dados inline, sem mock)
│   └── migrations/       # 0001_initial
├── templates/booking/    # base.html + páginas (home, club_detail, reserve_court...)
├── static/css|images/
└── venv/                 # não versionar; usar requirements.txt
```

## 6. Rotas (booking/urls.py)

`home` `/` · `signup` `/cadastro/` · `club_detail` `/clubes/<id>/` ·
`club_sport_courts` `/clubes/<id>/esportes/<sport>/quadras/` ·
`reserve_court` `/quadras/<id>/reservar/` · `my_reservations` `/minhas-reservas/` ·
`admin_dashboard` `/painel-admin/`.

**Refator de arquitetura (2026-07-02):** as telas magras `about`/`faq`/`contact` e a
lista `club_list` foram **removidas** — o conteúdo virou seções da **home** (busca ao
vivo + resultados + "como funciona" + FAQ + rodapé de contato). As rotas antigas
`/sobre` `/faq` `/contato` `/clubes` continuam existindo como **redirects** (RedirectView)
para as âncoras da home (`club_list` ainda é um nome de rota válido = redirect p/ `home`,
usado por `redirect("club_list")` nas views).

## 7. Como rodar (dev atual)

```bash
cd OnCourt
.\venv\Scripts\Activate.ps1      # PowerShell (Windows)
python manage.py migrate         # auth/sessions
python manage.py runserver
```

> **⚠️ Pegadinha de estáticos (CSS/JS).** Editar `static/css/styles.css` no fonte **não
> basta** para ver a mudança no navegador. Há WhiteNoise + `CompressedManifestStaticFilesStorage`:
> 1. **Cache do browser:** em DEBUG o `{% static %}` serve o arquivo SEM hash
>    (`/static/css/styles.css`), então o navegador entrega a versão antiga. **Hard refresh**
>    (`Ctrl+Shift+R`) resolve.
> 2. **Cópia coletada defasada:** o WhiteNoise serve a cópia hashada de `STATIC_ROOT`
>    (`staticfiles/css/styles.<hash>.css`). Rodar `python manage.py collectstatic --noinput`
>    para atualizar a cópia + o manifesto. **Obrigatório antes de deploy no Render.**
>
> **Sintoma típico:** HTML novo aparece (templates recarregam sozinhos) mas a página fica
> "crua"/sem estilo — parece que o CSS não foi escrito, mas o fonte está certo: é cache +
> cópia coletada velha.

## 8. Convenções de trabalho (do dono do projeto)

Derivado de `gpt_guideliine.md`. Em ordem de prioridade:

1. **Não chutar dados faltantes** — se faltar nome de coluna/schema/caminho/constraint,
   pare e pergunte (curto e numerado). Nunca assumir nomes de campos.
2. **Responder direto o que foi perguntado**, sem expandir para etapas futuras sozinho.
3. **Evolução incremental (MVP → reforços).** Simples primeiro; robustez (validação,
   logging, retries, edge cases) só quando pedido.
4. **Mudanças mínimas** em código existente; refactor amplo só com aprovação explícita.
5. **Ao concluir, ofereça opções de próximo passo** (gated) e espere a decisão.
6. **Trabalho de UI/UX:** seguir a skill `frontend-design` (evitar visual "templated";
   escolhas deliberadas de paleta/tipografia/layout). Usar `ui-ux-pro-max` para gerar
   design system / buscar estilos, paletas e pares de fonte (motor de busca offline em
   `~/.claude/skills/ui-ux-pro-max/scripts/search.py`). Usar `theme-factory` ao definir
   tema/cores. Validar mudanças de tela com `webapp-testing` (screenshot antes/depois,
   checar console) sempre que possível — requer Playwright instalado no venv.

### 8.1 Design system atual (2026-07-02) — "Dark Atlético + Volt"

- **Tema:** dark. Tokens em `static/css/styles.css` (`:root`): `--bg-deep #0E1116`,
  `--surface #171B22`, `--fg #F4F5F0`, acento **volt** `--accent #C7F94B`, status
  `--ok/--busy/--warn`. Sempre usar os tokens, nunca hex cru nos componentes.
- **Tipografia:** Barlow Condensed (títulos, `--font-display`) + Barlow (corpo) via
  Google Fonts (CDN, `display=swap`). Fallback `system-ui`.
- **Ícones:** SVG inline via `templates/booking/_icons.html`
  (`{% include ... with name="star" %}`). **Proibido emoji como ícone.**
- **Filtro de esporte → rótulo:** `{% load booking_extras %}` + `{{ valor|sport_label }}`
  (`booking/templatetags/booking_extras.py`).
- **Busca da home:** filtro client-side em `static/js/home-filter.js` (sem recarregar).
- **Imagens:** capas reais em `static/images/` (`hero-court.jpg`, `club-*.jpg`); clubes e
  quadras apontam para elas no banco. `default-court.svg` é só fallback.
- `LANGUAGE_CODE = pt-br` (localiza forms de auth e validadores de senha).

## 9. Pendências / riscos conhecidos

- [x] ~~`requirements.txt` inexistente.~~
- [x] ~~`SECRET_KEY` hardcoded e `DEBUG=True`.~~ → env-izado.
- [x] ~~Domínio só em memória / `mock_data.py`.~~ → schema relacional + ORM.
- [x] ~~Reservas efêmeras.~~ → persistidas em `Reservation`.
- [ ] Postgres real não conectado (Docker baixado, pendente reiniciar; ou usar Supabase).
- [ ] Sem testes automatizados (`booking/tests.py` vazio) — cobrir `availability.py`.
- [ ] Telas assumem `slot_minutes=60`; durações variáveis não foram exercitadas na UI.
- [ ] `forms.SPORT_CHOICES` é uma lista própria; idealmente derivar de `Sport.choices`.
- [ ] `README.md` praticamente vazio.

## 10. Operação / Deploy (Render + Supabase)

- **No ar:** https://oncourt-6zbu.onrender.com (URL fixa do serviço; o sufixo `-6zbu` só
  muda se o serviço for recriado). Detalhes/runbook em [`DEPLOY.md`](DEPLOY.md).
- **Auto-deploy:** ligado (*On Commit*). **`git push origin main` → redeploy automático**;
  commit local não dispara nada. Build que falha mantém a versão anterior no ar.
  Desligar/ajustar em Render → Settings (Auto-Deploy / Build Filters).
- **Build:** `bash ./build.sh` (pip install + collectstatic + migrate). **Start:**
  `gunicorn courton.wsgi:application`. O `seed_data` foi **removido do build** após a 1ª
  carga (rodava a cada deploy e sobrescrevia edições do admin); rodar manualmente via
  Render Shell quando precisar.
- **Env vars no Render:** `SECRET_KEY` (gerada), `DEBUG=False`, `ALLOWED_HOSTS=.onrender.com`,
  `CSRF_TRUSTED_ORIGINS=https://*.onrender.com`, `PYTHON_VERSION=3.13.4`, e o segredo
  `DATABASE_URL`.
- **Banco:** Supabase **Session pooler** (porta 5432, host `aws-1-us-west-2.pooler.supabase.com`,
  usuário `postgres.<ref>`). **Lição aprendida:** a conexão *Direct* do Supabase é
  **IPv6-only** e o Render não a alcança ("Network is unreachable") — sempre usar o pooler.
  Senha com caractere especial precisa ser **URL-encodada** na `DATABASE_URL` (`@` → `%40`).
- **Comandos de manutenção em produção** (ex.: `createsuperuser`, `seed_data --flush`):
  Render → **Shell**. Migrations também rodam sozinhas no build.
- **Segredos:** `.env` é local e gitignored — nunca commitar. Em produção tudo vem das
  env vars do Render.
- **Cold start:** Render free dorme após ~15 min sem acesso (~50s na 1ª request).
- **Próximo destino (prod AWS):** mesma codebase, trocar só `DATABASE_URL` p/ RDS e hospedar
  em App Runner/ECS (ou EC2).

---
> Source: [RodrigoFurukawa/OnCourt](https://github.com/RodrigoFurukawa/OnCourt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
