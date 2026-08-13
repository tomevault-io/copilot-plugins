## controle-de-acordos-de-cooperacao-tecnica-e-convenio-depi-fvs-rc

> **Projeto:** Painel de Controle de Acordos de Cooperação Técnica e Convênios — DEPI / FVS-RCP

# CLAUDE.md — Registro Histórico do Projeto

**Projeto:** Painel de Controle de Acordos de Cooperação Técnica e Convênios — DEPI / FVS-RCP
**Responsável:** Walter Oliva Pinto Filho Segundo
**Diretoria:** DEPI — Diretoria de Ensino, Pesquisa e Inovação / FVS-RCP
**Plataforma:** GitHub Pages (site estático) + GitHub Actions (automação de email mensal)

---

## Visão Geral

Painel web estático hospedado no GitHub Pages para monitoramento de vigência e renovação de instrumentos jurídicos (ACTs, Convênios e Termos de Colaboração) da DEPI/FVS-RCP. O sistema envia automaticamente, todo dia 1º de cada mês, um resumo por email para os gerentes responsáveis.

---

## Estrutura de Arquivos

```
/
├── index.html                        # Painel web (SPA estática, sem backend)
├── CLAUDE.md                         # Este arquivo — registro histórico
├── requirements.txt                  # Dependências Python (nenhuma externa)
├── .gitignore
├── assets/
│   ├── fvs-logo.png                  # Logo FVS-RCP
│   ├── fvs-logo_page-0001.jpg        # Logo FVS-RCP (alternativo)
│   └── logo_depi.jpeg                # Logo DEPI
├── data/
│   └── tbl_instrumentos.csv          # Base de dados dos instrumentos (editada manualmente)
├── scripts/
│   ├── monitor_act.py                # Lê o CSV e gera resumo_execucao.json
│   └── send_email.py                 # Lê o JSON e envia email via SMTP
└── .github/
    └── workflows/
        └── relatorio_mensal.yml      # GitHub Actions: roda dia 1º de cada mês
```

---

## Como o Sistema Funciona

### Painel Web (`index.html`)
- Página HTML única com JavaScript puro (sem framework)
- Lê `data/tbl_instrumentos.csv` via PapaParse (CDN)
- Gráficos via Plotly.js (CDN)
- Funcionalidades: KPIs, gráfico de status (donut), gráfico de vencimentos por mês (barra), mapa do Brasil por UF, gráfico de diretorias, tabela com filtros e busca, detalhamento expandível por instrumento
- Classificação de vigência:
  - **VIGENTE**: termina em mais de 180 dias
  - **ALERTA 180 DIAS**: termina em até 180 dias
  - **SEM DATA**: sem data de vencimento cadastrada
  - **ARQUIVADO**: oculto do painel (campo `arquivado = SIM`)

### Automação Mensal (GitHub Actions)
Executa em dois passos sequenciais:
1. `monitor_act.py` — lê o CSV, classifica os instrumentos em faixas de 60 e 180 dias, salva `data/resumo_execucao.json`
2. `send_email.py` — lê o JSON e envia o resumo por email via SMTP

**Classificação do email (diferente do painel):**
- Confortável: > 180 dias
- Atenção: 61–180 dias
- Crítico: ≤ 60 dias
- Vencido: data já passou
- Sem data: sem registro de vigência

---

## Configuração de Secrets no GitHub

Todos os parâmetros sensíveis são gerenciados via **GitHub Settings → Secrets and variables → Actions**:

| Secret       | Descrição                                              | Obrigatório |
|--------------|--------------------------------------------------------|-------------|
| `SMTP_USER`  | Email remetente (login no servidor SMTP)               | Sim         |
| `SMTP_PASS`  | Senha de app (não a senha normal da conta)             | Sim         |
| `SMTP_TO`    | Emails destinatários, separados por vírgula            | Sim         |
| `SITE_URL`   | URL completa do painel (ex: https://usuario.github.io/...) | Sim     |
| `SMTP_HOST`  | Servidor SMTP (padrão: smtp.gmail.com)                 | Não         |
| `SMTP_PORT`  | Porta SMTP (padrão: 587)                               | Não         |

### Como gerar senha de app no Gmail
1. Acesse sua conta Google → Segurança → Verificação em duas etapas (deve estar ativa)
2. Em "Senhas de app", crie uma nova para "Email" / "Outro"
3. Use a senha gerada como `SMTP_PASS`

---

## Como Atualizar a Base de Dados

1. Edite `data/tbl_instrumentos.csv` diretamente no GitHub (aba "Edit this file") ou localmente e faça push
2. O painel atualiza automaticamente (sem rebuild — é lido em tempo real)
3. O email mensal usará os dados na próxima execução do dia 1º

### Colunas do CSV

| Coluna                  | Descrição                                              |
|-------------------------|--------------------------------------------------------|
| `id_instrumento`        | Número sequencial interno                              |
| `tipo_instrumento`      | ACT / Convênio / TC                                    |
| `identificacao`         | Número do instrumento (ex: 20/2023)                    |
| `instituicao_parceira`  | Nome da instituição parceira                           |
| `diretoria_responsavel` | Diretoria da FVS-RCP responsável                       |
| `gerencia_responsavel`  | Gerência responsável                                   |
| `numero_processo`       | Número do processo SEI                                 |
| `vigencia_inicio`       | Data de início (formato DD/MM/AAAA)                    |
| `vigencia_termino`      | Data de término (formato DD/MM/AAAA) — campo crítico   |
| `arquivado`             | SIM ou NÃO — se SIM, oculta do painel                  |
| `possui_aditivo`        | SIM ou NÃO                                             |
| `quantos_aditivos`      | Quantidade de aditivos (número)                        |
| `data_assinatura`       | Data de assinatura do instrumento                      |
| `data_publicacao`       | Data de publicação no DOE                              |
| `publicacao_doe`        | Número do extrato publicado no DOE                     |
| `link_publicacao_doe`   | URL da publicação no DOE (se disponível)               |
| `coordenador_parceiro`  | Nome e email do coordenador na instituição parceira    |
| `coordenador_fvsrcp`    | Nome e email do responsável na FVS-RCP                 |
| `repasse_financeiro`    | SIM ou NÃO                                             |
| `objetivo_resumido`     | Descrição resumida do objeto do instrumento            |
| `produtos_gerados`      | Produtos e entregas (separados por `;`)                |
| `estado`                | Estado da instituição parceira                         |
| `uf`                    | UF da instituição parceira (sigla de 2 letras)         |
| `status_execucao`       | Ex: "Em Andamento", "Concluído"                        |

---

## Histórico de Alterações

### Sessão 3 — 21/03/2026 (Walter + Claude Code) — Redesign do rodapé

**Mudanças no `index.html`:**

1. **Rodapé reformulado** — substituído o estilo anterior (card interno com borda/sombra) pelo novo modelo institucional centralizado
2. **Estrutura HTML do rodapé** — agora usa `.site-footer-inner` como container centrado com `max-width: 1100px`; título em `.footer-title`; divisor decorativo em `.footer-divider`; linhas de texto em `<p>`
3. **CSS atualizado** — removidas as classes `.footer-brand` e `.footer-address`; adicionadas `.site-footer-inner`, `.footer-title`, `.footer-divider`
4. **Cor de fundo** — fundo branco (`#fff`) com texto escuro (`#333`); divisor em `rgba(0,0,0,.15)` (em vez do verde `#5f7f73` do modelo original)

---

### Sessão 2 — 21/03/2026 (Walter + Claude Code) — Melhorias visuais

**Mudanças no `index.html`:**

1. **KPI cards coloridos** — "Vigentes" com borda/valor verde, "Atenção" em âmbar, "Crítico" em vermelho
2. **Novo card KPI "Crítico (≤60 dias)"** — separado do card de atenção
3. **`computeStatus`** — agora retorna três faixas: VIGENTE / ALERTA 180 DIAS / CRÍTICO 60 DIAS (alinhado ao email)
4. **Filtro de status** — adicionada opção "CRÍTICO (≤60 dias)"
5. **Legenda do gráfico de status** — adicionado item vermelho para CRÍTICO
6. **Gráfico donut corrigido** — removido `textinfo: "none"` que suprimia os valores; agora exibe valor e % dentro das fatias; CRÍTICO adicionado ao gráfico em vermelho
7. **Botão da tabela** — "Ver"/"Fechar" substituído por ▼/▲
8. **Rodapé institucional** adicionado com nome da diretora, nome da diretoria e endereço

---

### Sessão 1 — 21/03/2026 (Walter + Claude Code)

**Contexto:** O projeto foi inicialmente hospedado com o email pessoal `walterolivafvs@gmail.com`. Foi necessário migrar para o email institucional da diretoria DEPI/FVS-RCP.

**Erros corrigidos:**

1. **`data/tbl_instrumentos.csv` — id=5 (ACT 30/2023 / FIOCRUZ):**
   - Typo no nome: `Ségio Luiz Bessa Luz` → `Sérgio Luiz Bessa Luz` (faltavam "r" e acento em "é")
   - Parêntese duplo no email: `sergio.luz@fiocruz.br))` → `sergio.luz@fiocruz.br)`

2. **`data/tbl_instrumentos.csv` — id=9 (ACT 34/2023 / FIOCRUZ):**
   - Número de processo com prefixo duplicado por erro de digitação:
   - `01.02.017306.00001.02.017306.003246/2024-75` → `01.02.017306.003246/2024-75`

3. **`scripts/send_email.py`:**
   - Removido comentário com email pessoal exposto no código (linha 47)
   - URL do painel (GitHub Pages) deixou de ser hardcoded no código-fonte
   - Passou a ser configurável via secret `SITE_URL` no GitHub

4. **`.github/workflows/relatorio_mensal.yml`:**
   - Adicionadas variáveis de ambiente: `SITE_URL`, `SMTP_HOST`, `SMTP_PORT`
   - Permite configurar servidor SMTP e URL do painel sem alterar o código

**Para completar a migração (ações necessárias no GitHub):**

Acesse o repositório institucional → **Settings → Secrets and variables → Actions** e configure:

| Secret       | Valor a definir                                        |
|--------------|--------------------------------------------------------|
| `SMTP_USER`  | Email institucional da diretoria (ex: depi@fvs.am.gov.br) |
| `SMTP_PASS`  | Senha de app gerada para o novo email                  |
| `SMTP_TO`    | Emails dos gerentes (separados por vírgula)            |
| `SITE_URL`   | Nova URL do painel (ex: https://novouser.github.io/Controle_de_Acordos_de_Cooperacao_Tecnica_e_Convenio_DEPI_FVS_RCP/) |
| `SMTP_HOST`  | Se o email institucional não for Gmail, definir o servidor SMTP correto |

**Sugestões de melhoria identificadas e implementadas na Sessão 2:**

- KPI cards com cores (verde/âmbar/vermelho) — implementado
- Novo KPI "Crítico (≤60 dias)" separado de "Atenção (61–180 dias)" — implementado
- `computeStatus` agora retorna "CRÍTICO 60 DIAS" para instrumentos com ≤60 dias, alinhando painel ao email
- Botão "Ver/Fechar" substituído por ▼/▲
- Gráfico donut corrigido: `textinfo: "none"` removido; agora exibe valores e percentuais dentro das fatias
- Rodapé institucional adicionado com endereço DEPI/FVS-RCP

**Observações pendentes:**

- O `.DS_Store` dentro de `data/` pode estar sendo rastreado pelo Git se foi adicionado antes do `.gitignore`. Para remover: `git rm --cached data/.DS_Store`
- O arquivo `requirements.txt` contém apenas um comentário — funcional, mas pode ser removido.

---

## Tecnologias Utilizadas

- **Frontend:** HTML5 + CSS3 + JavaScript puro (sem framework)
- **Gráficos:** [Plotly.js 2.35.2](https://plotly.com/javascript/) via CDN
- **Parsing CSV:** [PapaParse 5.4.1](https://www.papaparse.com/) via CDN
- **Hospedagem:** GitHub Pages (estático)
- **Automação:** GitHub Actions (Ubuntu)
- **Email:** Python `smtplib` + `email.message.EmailMessage` (SMTP / STARTTLS)
- **Processamento:** Python 3.11

---

## Estrutura Organizacional (FVS-RCP)

**Diretorias**
- `DEPI` — Diretoria de Ensino, Pesquisa e Inovação
- `DVACD` — Diretoria de Vigilância Ambiental e Controle de Doenças
- `DVE` — Diretoria de Vigilância Epidemiológica
- `LACEN` — Laboratório Público Central

**Gerências**
- `GPAVS` — Gerência de Pesquisa Aplicada a Vigilância em Saúde
- `GRNB` — Gerência de Riscos Não Biológicos
- `GVDANT` — Gerência de Vigilância de Doenças e Agravos Não Transmissíveis
- `PECT` — Programa Estadual de Controle da Tuberculose
- `SGENTO` — Subgerência de Entomologia
- `SASS` — Sala de Análise de Situação de Saúde

---

## Atualização — 23/03/2026 (Sessão 4 — Walter + Claude Code) — Ajuste visual da marca FAROL

### Alterações realizadas

1. **Posicionamento da marca FAROL corrigido** — o bloco `#farol-assinatura-fixa` passou de centralizado verticalmente no rodapé (`top: 50%; transform: translateY(-50%)`) para ancorado na borda inferior direita (`bottom: 10px; right: 16px`), deixando o ícone e o texto "FAROL" na extremidade inferior direita da página.

2. **Cor da marca FAROL alterada para cinza médio** — tanto o stroke do SVG quanto a cor do texto `.farol-texto` foram alterados de tons escuros (`#111` e `#555`) para `#888` (cinza médio), reduzindo o contraste e conferindo um aspecto mais discreto à assinatura.

### Arquivos afetados

| Arquivo | Tipo de alteração |
|---|---|
| `index.html` | CSS do `#farol-assinatura-fixa` (posição) e atributo `stroke` do SVG + cor do texto (cor) |
| `CLAUDE.md` | Adição desta entrada de histórico |

---

## Atualização — 23/03/2026

### Alterações realizadas

1. **Título do painel atualizado** — o `<title>` no `<head>` e o `<h1 class="title">` no cabeçalho foram alterados de "Controle de Acordos de Cooperação Técnica e Convênios" para "Controle de Acordos de Cooperação Técnica, Convênios e Termos de Colaboração", refletindo o escopo real dos instrumentos gerenciados.

2. **Criação da marca visual FAROL** — implementada como assinatura institucional do sistema. Após múltiplas iterações de posicionamento, a solução definitiva adotada foi:
   - SVG vetorial minimalista representando um farol (base trapezoidal, corpo vertical, cabine superior, cúpula em arco e dois feixes diagonais de luz)
   - Texto "FAROL" abaixo do ícone
   - Bloco HTML inserido dentro do `<footer class="site-footer">`, fora do `.site-footer-inner`
   - Posicionamento via `position: absolute` ancorado ao `<footer>` (que recebeu `position: relative`), com `right: 28px` e `top: 50%; transform: translateY(-50%)`
   - Identificador exclusivo `id="farol-assinatura-fixa"` para evitar conflitos de especificidade CSS
   - Oculto em telas menores que 600px via `@media`

3. **Tentativas anteriores de `position: fixed` descartadas** — foram realizadas quatro tentativas de posicionamento fixo na viewport (flutuando sobre a página). Todas falharam no ambiente de uso (o elemento acompanhava a rolagem). A causa técnica identificada foi incompatibilidade do `position: fixed` com o contexto de renderização — possivelmente iframe, WebView ou zoom aplicado pelo sistema operacional. A solução definitiva substituiu `fixed` por `absolute` dentro do rodapé.

4. **Ajustes colaterais revertidos** — durante as tentativas de correção do `fixed`, foram introduzidas e posteriormente revertidas as seguintes alterações em `html`/`body`:
   - `html { overflow-y: scroll }` — introduzido e revertido (causava quebra do `fixed` em Safari/WebKit)
   - `body { overflow-y: visible }` — mantido após reversão
   - Estado final: `html { scrollbar-gutter: stable; }` e `body { overflow-y: visible; }`

### Justificativa técnica

- O título "Termos de Colaboração" foi adicionado porque o CSV e o painel já gerenciam esse tipo de instrumento (`tipo_instrumento = TC`), e a ausência do termo no título criava inconsistência institucional.
- A marca FAROL foi criada como identidade visual do sistema de monitoramento, reforçando conceitualmente a função de orientação estratégica e vigilância contínua da DEPI sobre os instrumentos jurídicos.
- A opção por `position: absolute` no rodapé foi a única tecnicamente confiável no ambiente identificado, pois não depende de comportamento de viewport e é imune a contextos de rendering que quebram `position: fixed`.

### Impacto no sistema

- **Layout:** nenhum impacto no cabeçalho, sidebar, KPIs, gráficos ou tabela. O rodapé institucional permanece com o mesmo conteúdo centralizado; a marca FAROL é adicionada à direita sem deslocar o texto central.
- **Dados:** nenhum impacto. Nenhum script, filtro ou lógica de negócio foi alterado.
- **Usabilidade:** a marca FAROL é puramente decorativa (`aria-hidden="true"`, `pointer-events: none`) e não interfere em nenhuma interação do usuário.
- **Responsividade:** em telas com largura ≤ 600px, a marca é ocultada automaticamente para não comprimir o conteúdo do rodapé.

### Arquivos afetados

| Arquivo | Tipo de alteração |
|---|---|
| `index.html` | Título (`<title>` e `<h1>`), CSS do rodapé e da marca FAROL, HTML do rodapé com o bloco `#farol-assinatura-fixa` |
| `CLAUDE.md` | Adição desta entrada de histórico |

### Recomendações futuras

- **Avaliar uso de `position: sticky` no rodapé** — se o objetivo futuro for manter a marca FAROL sempre visível na parte inferior da janela sem depender de `fixed`, considerar um layout com `min-height: 100vh` no `<body>` e `margin-top: auto` no `<footer>`, o que manteria o footer sempre ao fundo mesmo em páginas com pouco conteúdo.
- **Formalizar o nome FAROL** — se o sistema for oficialmente nomeado "FAROL", atualizar também o `<title>`, o subtítulo do painel e eventuais metadados Open Graph para consistência institucional.
- **Documentar tipo de instrumento TC** — o `CLAUDE.md` ainda descreve o projeto no título como "Controle de Acordos de Cooperação Técnica e Convênios"; considerar atualizar também o cabeçalho deste arquivo para incluir "Termos de Colaboração".
- **Testar em diferentes ambientes de visualização** — dado que `position: fixed` não funcionou conforme esperado, recomenda-se registrar o ambiente exato de uso (navegador, versão, zoom, se há iframe ou WebView) para diagnóstico definitivo em futuras sessões.

---
> Source: [DEPI-FVS-RCP/Controle_de_Acordos_de_Cooperacao_Tecnica_e_Convenio_DEPI_FVS_RCP](https://github.com/DEPI-FVS-RCP/Controle_de_Acordos_de_Cooperacao_Tecnica_e_Convenio_DEPI_FVS_RCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
