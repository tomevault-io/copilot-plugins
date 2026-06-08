## am-converte

> Operação da AM Converte. Aqui ficam clientes, estratégias, conteúdos, propostas e entregas da agência de consultoria comercial jurídica.

# AM Converte — MazyOS

Operação da AM Converte. Aqui ficam clientes, estratégias, conteúdos, propostas e entregas da agência de consultoria comercial jurídica.

**Estrutura de pastas:**
- `_memoria/` — quem é a agência, como falamos, foco atual
- `identidade/` — referências de marca (em construção)
- `clientes/` — uma subpasta por cliente, autossuficiente
- `propostas/` — propostas em andamento
- `marketing/` — conteúdo institucional da AM Converte
- `saidas/` — documentos pontuais, análises
- `dados/` — arquivos a analisar (relatórios de cliente, exports)
- `scripts/` — automações e scripts de apoio

---

## Sobre a AM Converte

Somos uma consultoria comercial jurídica. Atendemos advogados — principalmente trabalhistas e previdenciários — que já recebem leads mas não convertem, ou que precisam de tráfego pago para gerar demanda. Entregamos estrutura comercial completa: processo, script, closers e gestão.

**Time:**
- **Álvaro Bastos** — estrategista e gestor de tráfego
- **Gabriel Akira** — closer
- **Valéria** — closer
- **Carol Damaceno** — social media, contratos e financeiro

**Serviços principais:**
- Estruturação do comercial do escritório
- Gestão de tráfego pago (Meta Ads)
- Closers dedicados pra converter leads em contratos

---

## Contexto do negócio

No início de toda conversa, ler os seguintes arquivos (quando existirem e estiverem preenchidos):

1. `_memoria/empresa.md` — quem somos, o que fazemos, equipe
2. `_memoria/preferencias.md` — tom de voz, estilo de escrita, o que evitar
3. `_memoria/estrategia.md` — foco atual, prioridades, prazos

Usar essas informações como base pra qualquer resposta ou decisão. Ao sugerir prioridades, formatos ou abordagens, considerar o foco atual descrito em `estrategia.md`.

Pra qualquer tarefa visual (carrossel, post, proposta), consultar `identidade/design-guide.md` como referência de estilo.

Não é necessário listar o que foi lido nem confirmar a leitura. Apenas usar o contexto naturalmente.

---

## Fluxo de trabalho

Antes de executar qualquer tarefa, verificar se existe skill relevante em `.claude/skills/`. Se encontrar, seguir as instruções da skill. Se não encontrar, executar a tarefa normalmente.

Ao concluir uma tarefa que não tinha skill mas parece repetível, perguntar:

> "Isso pode virar uma skill pra próxima vez. Quer que eu crie?"

Não perguntar pra tarefas pontuais ou perguntas simples. Só quando o padrão de repetição for claro.

---

## Aprender com correções

Quando o usuário corrigir algo, melhorar uma resposta ou dar uma instrução que parece permanente (frases como "na verdade é assim", "não faça mais isso", "prefiro assim", "sempre que...", "evita...", "da próxima vez..."), perguntar:

> "Quer que eu salve isso pra não precisar repetir?"

Se sim, identificar onde faz mais sentido salvar:

- **Sobre o negócio** (clientes, serviços, mercado) → `_memoria/empresa.md`
- **Sobre preferências e estilo** (tom de voz, formato, o que evitar) → `_memoria/preferencias.md`
- **Sobre prioridades e foco** (projetos, metas, prazos) → `_memoria/estrategia.md`
- **Regra de comportamento nessa pasta** → próprio `CLAUDE.md`

Salvar com uma linha nova clara, sem reformatar o arquivo inteiro. Confirmar mostrando a linha adicionada.

---

## Manter contexto atualizado

Ao terminar uma tarefa que mudou algo relevante (cliente novo, skill nova, mudança de foco, processo novo, ferramenta instalada), perguntar:

> "Isso mudou algo no teu contexto. Quer que eu atualize a memória?"

Se sim, identificar o que atualizar:

- **Cliente, serviço, ferramenta, equipe** → `_memoria/empresa.md`
- **Mudança de prioridade ou foco** → `_memoria/estrategia.md`
- **Tom ou estilo** → `_memoria/preferencias.md`
- **Pasta, regra de organização, skill criada** → `CLAUDE.md`
- **Visual (cores, fontes, logo)** → `identidade/design-guide.md`

Mostrar o que vai mudar antes de salvar. Não reformatar o arquivo inteiro.

**Quando NÃO perguntar:** tarefas pontuais, perguntas simples, mudanças já salvas.

**Dica:** rode `/atualizar` pra uma varredura completa quando houver dúvida.

---

## Regras da operação

- Cliente novo → criar pasta `clientes/<Nome>/` com briefing, estratégia e subpastas conforme as entregas contratadas
- Proposta nova → `propostas/<cliente>-<data>.html` antes de fechar
- Casos de sucesso ficam em `clientes/<Nome>/caso.md` (reuso em pitches)
- Qualquer rotina que o Álvaro repetir mais de uma vez → candidata a skill via `/mapear-rotinas`

---

## Criação de skills

Quando o usuário pedir skill nova:

1. Verificar se existe template relevante em `templates/skills/`. Se existir, usar como base e adaptar
2. Perguntar se é específica desse projeto ou útil em qualquer:
   - Específica → `.claude/skills/nome-da-skill/SKILL.md` (local)
   - Universal → `~/.claude/skills/nome-da-skill/SKILL.md` (global)
3. Ler `_memoria/empresa.md` e `_memoria/preferencias.md` pra calibrar o conteúdo ao contexto
4. Seguir o fluxo da skill-creator nativa do Claude Code

---

## Ferramentas conectadas

- [ ] Meta Ads
- [ ] Google Ads
- [ ] Notion
- [ ] Gmail
- [ ] Google Calendar
- [ ] Canva

*(Marcar conforme for instalando os MCPs)*

---
> Source: [eualvarobastos/am-converte](https://github.com/eualvarobastos/am-converte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
