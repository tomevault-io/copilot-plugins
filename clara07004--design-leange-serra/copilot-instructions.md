## design-leange-serra

> > **Este projeto é uma base/template.** Não é executado diretamente aqui — projetos reais são replicados a partir deste. Credenciais, contexto (`_contexto/`) e outputs (`conteudo/`) ficam no projeto replicado, não aqui. Nunca tentar testar, executar scripts ou verificar configurações neste workspace.

# CCOS — Sistema de Automação de Conteúdo

## O que é esse workspace

> **Este projeto é uma base/template.** Não é executado diretamente aqui — projetos reais são replicados a partir deste. Credenciais, contexto (`_contexto/`) e outputs (`conteudo/`) ficam no projeto replicado, não aqui. Nunca tentar testar, executar scripts ou verificar configurações neste workspace.

Sistema de automação do processo de criação de conteúdo para redes sociais. Orquestra skills de IA para ir da definição estratégica até a entrega do pacote de conteúdo pronto para publicação.

**Empresa:** [Preencher com o nome e descrição da empresa após rodar `/setup`]

**Estrutura de pastas:**
- `_contexto/` — memória do sistema (não apagar)
- `marca/` — DESIGN.md e identidade visual
- `dados/` — arquivos para análise (CSV, PDF, prints, referências)
- `templates/skills/` — templates de skills prontos pra personalizar com /mapear
- `templates/ferramentas/catalogo.md` — APIs e ferramentas disponíveis pra usar em skills
- `credentials/` — credenciais e tokens (não committar)

---

## Contexto do negócio

No início de toda conversa, ler os seguintes arquivos (se existirem e estiverem configurados):

1. `_contexto/empresa.md` — quem é o usuário, o que faz, como funciona o negócio
2. `_contexto/preferencias.md` — tom de voz, estilo de escrita, o que evitar
3. `_contexto/estrategia.md` — foco atual, prioridades, o que pode esperar
4. `_contexto/referencias.md` — pastas do Google Drive com material de referência visual

Usar essas informações como base pra qualquer resposta ou decisão. Ao sugerir prioridades, formatos ou abordagens, considerar o foco atual descrito em `estrategia.md`.

Para qualquer tarefa visual (carrossel, proposta, slide, landing page), consultar `marca/DESIGN.md` como referência de estilo.

**Referências visuais do Drive:** quando `_contexto/referencias.md` tiver pastas configuradas, consultá-las antes de criar qualquer conteúdo visual. Usar o MCP do Google Drive para listar os arquivos da pasta relevante (`search_files` com `parentId = 'ID_DA_PASTA'`) e baixar as imagens com `download_file_content`. Priorizar imagens menores que 300KB para caber no contexto. Usar o material encontrado como referência de estilo, produto e padrão visual — não como template a copiar literalmente.

Não é necessário listar o que foi lido nem confirmar a leitura. Apenas usar o contexto naturalmente.

---

## Conhecimento da pousada (consulta obrigatória)

**Antes de gerar calendário, briefing, roteiro, carrossel ou post estático**, ler os arquivos relevantes em `pousada/`. Esse diretório consolida toda a documentação oficial da LeAnge (unidades, suítes, gastronomia, regras de hospedagem, política de pets, experiência).

A LeAnge tem um posicionamento definido em `_contexto/empresa.md` — o conteúdo precisa:

- **Ser realista e verdadeiro:** nunca inventar capacidades, valores, comodidades ou regras. Tudo precisa estar lastreado em `pousada/`
- **Conter informação real da pousada:** citar suítes, espaços, serviços e políticas exatamente como documentados
- **Permitir que o hóspede conheça a experiência pelo post:** o conteúdo é parte da qualificação do lead, não decoração visual
- **Manter o posicionamento:** vocabulário alinhado com `_contexto/preferencias.md`, sem promessas genéricas sem lastro

Arquivos de consulta principais:
- `pousada/README.md` — índice de todos os tópicos
- `marca/identidade-visual.md` — grafia da marca, paleta, tipografia, logos
- `pousada/fotos-unidades/` — acervo visual real das unidades (priorizar sobre IA quando possível)
- `fotos serra/` — biblioteca de 608 fotos reais catalogadas. **Antes de gerar qualquer imagem, PERGUNTAR à Paola se ela quer uma foto da biblioteca ou uma foto própria (fora do acervo)** — ver regra obrigatória na Etapa 2. Se ela escolher a biblioteca, achar a foto com o buscador: `python "fotos serra/buscar-fotos.py" <palavras> --json --limit 8` (retorna caminho do arquivo + descrição, ranqueado; busca sem acento). Sem termos = estatísticas. Catálogo completo: `fotos serra/catalogo-imagens.csv`.
- Demais arquivos em `pousada/` conforme indexado no README

**Frases genéricas para revisar:** "qualidade superior", "experiência única", "conforto incomparável" sem lastro concreto = reescrever com o diferencial real documentado (ex.: sem limite de porte/raça, pet solto no restaurante e piscina, all inclusive com restrições alimentares atendidas).

**Grafia obrigatória da marca:** sempre `LeAnge` (L e A maiúsculos, sem espaço, sem hífen). Nunca "Le Ange", "LE ANGE", "leange". Detalhe em [marca/identidade-visual.md](marca/identidade-visual.md).

**REGRA DE ESCOPO — SOMENTE LeAnge Serra:** o grupo tem duas unidades (Serra, em Miguel Pereira, e Mar, em Búzios), mas **este repositório é focado exclusivamente na LeAnge Serra**. Se qualquer documento, mensagem ou fonte trouxer menção ou informação sobre a **LeAnge Mar**, você deve **ignorar** — **JAMAIS** documentar informação da Mar e **JAMAIS misturar** dados das duas pousadas. Todo conteúdo, dado, comodidade, regra e visual documentado aqui é da Serra.

**REGRA DE COPY — preços/valores NUNCA entram no criativo:** preços, diárias, valores, descontos, taxas, percentuais de reembolso/cancelamento e condições comerciais (late check-out, early check-in, cortesias, brindes) **NUNCA** aparecem em copy, legenda, roteiro ou peça — **exceto** quando a Paola pedir explicitamente uma condição/valor específico para um criativo. Detalhe em [_contexto/preferencias.md](_contexto/preferencias.md).

---

## Skills disponíveis

**Skills do sistema CCOS (genéricas):**
- `/setup` — configura o sistema pro seu negócio (rodar na primeira vez)
- `/syncar` — salva o trabalho no GitHub (commit + push)
- `/mapear` — entrevista processos repetitivos e cria skills personalizadas

**Skills de conteúdo:**
- `/calendario-comercial` — mapa de oportunidades do período (quando e o quê postar)
- `/briefing-leange` — briefing completo de um tema: objetivo, mensagem, formato, referências
- `/carrossel-leange` — produção de carrossel: texto + HTML + PNG via Playwright
- `/estatico-leange` — produção de post card único: foto IA + HTML + PNG via Playwright
- `/roteiro-leange` — roteiro de vídeo para Reels/TikTok (orgânico via Ogilvy, tráfego via Schwartz)
- `/publicar-social-leange` — publica conteúdo aprovado no Instagram, TikTok, LinkedIn

**Skills de pesquisa e ideação:**
- `/gerador-de-angulos-para-um-tema` — 10 lentes criativas para explorar um tema antes do briefing
- `/gerador-de-angulos-de-conteudo` — matrix perspectivas × audiência × formatos narrativos, 10 ângulos únicos
- `/banco-de-objecoes-do-avatar` — mapeia 6 tipos de objeção por ICP com resposta em conteúdo

**Skills de copy e distribuição:**
- `/hooks-para-carrossel` — 5 opções de capa para carrossel com direção visual (usar antes do /carrossel-leange)
- `/hooks-para-instagram-reels` — 7 tipos de hook combinados (primeiro frame + frase de abertura)
- `/legenda-para-carrossel` — legenda orientada a save com CTA específico
- `/legenda-para-reel` — legenda que complementa o vídeo sem repetir o script
- `/legenda-para-post-estatico` — 4 tipos de legenda para post estático
- `/carrossel-de-quebra-de-objecao` — carrossel de fundo de funil em 3 movimentos (nomeação → reframe → prova)
- `/1-conteudo-em-7-formatos` — repurposing: transforma 1 conteúdo em Reel, Carrossel, Story, Thread, LinkedIn, E-mail e Post estático

**Skills de imagem:**
- `/gerador-de-prompts-de-imagem` — prompt estruturado para gpt-image-1 (usar antes de gerar imagem)
- `/gerador-de-prompts-para-imagens-da-pousada` — prompts para as estéticas de produto da empresa (estilos configurados em `marca/DESIGN.md`)

**Motores (usados internamente pelas skills de produção):**
- `/gpt-image2-leange` — gera foto de fundo via GPT Image 2 (motor de imagem do carrossel e post estático)
- `/ogilvy-copy` — copy de marca e conteúdo orgânico (motor do roteiro orgânico)
- `/schwartz-copy` — copy de resposta direta (motor do roteiro de tráfego pago)

---

## Fluxo principal de conteúdo

### Etapa 1 — Ideação e planejamento (sempre)

```
/gerador-de-angulos-para-um-tema   ← 10 lentes criativas para um tema
    ou
/gerador-de-angulos-de-conteudo    ← matrix perspectivas × audiência × formatos
    ↓ [escolhe ângulo]
/calendario-comercial              ← quando e o quê postar
    ↓ [aprova calendário]
/briefing-leange                    ← define objetivo, mensagem e formato
    ↓ [aprova briefing]
```

**REGRA OBRIGATÓRIA — Calendário de novo mês entrega SEMPRE 3 arquivos**

Toda vez que rodar `/calendario-comercial` para um mês (ou qualquer período fechado), gerar dentro de `conteudo/calendarios/[periodo]/`:

1. **`calendario-detalhado.md`** — post a post, numerado, com tema/formato/janela/status (alimenta os `/briefing-leange` posteriores)
2. **`_aprovado.md`** — memória da aprovação: tema narrativo, mix, apagões, picos, ajustes feitos, o que evitar
3. **`dashboard.html`** — grid visual do mês inteiro, derivado direto do `calendario-detalhado.md`, usando o template `templates/dashboard-calendario.html` (identidade visual da marca, canvas 1920×1080)

O dashboard **não é opcional** — é entrega obrigatória junto com o calendário, mesmo que o usuário não peça explicitamente. Após gerar o HTML, perguntar se renderiza o PNG via Playwright (esse passo sim é opcional).

A skill `/calendario-comercial` já tem a especificação completa na seção "8. ENTREGAS OBRIGATÓRIAS" — seguir literalmente.

---

### Etapa 2 — Produção (escolher o fluxo pelo formato)

**REGRA OBRIGATÓRIA — Confirmar o formato antes de produzir (exceto carrossel)**

Quando o conteúdo de um dia estiver marcado no calendário como **Reel, post estático, vídeo ou qualquer formato que NÃO seja carrossel**, SEMPRE perguntar ao usuário qual o formato do conteúdo antes de gerar qualquer coisa (hook, roteiro, briefing, prompt, imagem, HTML). Não assumir o formato do calendário automaticamente — ele é sugestão, a decisão final é do usuário. Quando o formato for **carrossel**, seguir direto o fluxo sem perguntar.

**REGRA OBRIGATÓRIA — Origem da imagem: perguntar ANTES de gerar**

Sempre que uma peça precisar de imagem/foto (carrossel, post estático, capa, slide, story), **antes de gerar qualquer imagem por IA, perguntar à Paola qual a origem da foto:**

1. **Foto da biblioteca** (`fotos serra/`) — buscar com `python "fotos serra/buscar-fotos.py" <palavras> --json --limit 8`, apresentar as opções ranqueadas e usar a que ela escolher.
2. **Foto própria dela** (fora da biblioteca) — aguardar ela enviar/apontar o arquivo.

Só partir para geração por IA (`/gpt-image2-leange` etc.) se a Paola disser explicitamente que não quer foto real (nem da biblioteca, nem própria). Priorizar sempre foto real — nunca gerar imagem por IA sem antes fazer essa pergunta.

**REGRA OBRIGATÓRIA — Nomenclatura da pasta de produção pela DATA DE PUBLICAÇÃO**

A pasta de produção de cada conteúdo é nomeada pela **data de publicação** no formato `dia-DD-tema-curto` (ex.: conteúdo que publica em 17/07 → `dia-17-serra-inverno-lareira`). **Nunca usar o número do post do calendário** — em meses com domingos sem post ou apagões, o número do post deixa de coincidir com o dia do mês (ex.: Post 15 publica em 17/07). O `DD` é sempre o dia em que o conteúdo vai ao ar.

#### Carrossel

**FLUXO PRINCIPAL ★ — usar SEMPRE este, a menos que o usuário peça explicitamente outro.** Quando o calendário já existe, começa no `/briefing-leange` (não nos hooks):
```
/briefing-leange                  ← ponto de partida (calendário já aprovado)
    ↓ [aprova briefing]
/gerador-de-prompts-de-imagem    ← FLUXO PRINCIPAL ★ — prompts da capa + de cada slide
    ↓ [aprova os prompts]
/gpt-image2-leange                ← gera todas as imagens (capa + slides)
    ↓ [aprova as imagens]
/carrossel-leange                 ← monta os HTMLs + renderiza PNG
    ↓ [aprova]
/legenda-para-carrossel
    ↓ [aprova o conteúdo final]
/publicar-social-leange           ← publicação
```

**Fluxo rápido (alternativa) — só quando o usuário pedir, ou para algo pontual sem controle granular de imagem:** `/carrossel-leange` cuida de tudo internamente (texto + prompt + imagem IA + HTML + PNG):
```
/carrossel-leange        ← texto + geração de imagem + HTML + PNG (tudo em um)
    ↓
/legenda-para-carrossel
```

> `/hooks-para-carrossel` é skill **auxiliar opcional** — usar só quando o usuário quiser explorar opções de capa antes. Não é etapa do fluxo principal.

**Observação:** quando houver fotos reais de produto disponíveis no Google Drive (`_contexto/referencias.md` → pasta "Fotos do Produto"), priorizá-las em vez de gerar imagem IA. Buscar via MCP Drive (`search_files` com o ID da pasta configurado em `_contexto/referencias.md`), baixar com `download_file_content`, extrair via Python e salvar como `img-slideXX.jpg` na pasta do carrossel. Resultado superior ao de qualquer geração IA.

---

#### Post estático

```
/gerador-de-prompts-para-imagens-da-pousada   ← ou /gerador-de-prompts-de-imagem
    ↓ [aprova prompt]
/gpt-image2-leange                             ← gera foto de fundo
    ↓ [aprova imagem]
/estatico-leange                               ← monta HTML + renderiza PNG
    ↓
/legenda-para-post-estatico
```

---

#### Vídeo (Reels / TikTok)

```
/hooks-para-instagram-reels   ← hook do primeiro frame + frase de abertura
    ↓ [escolhe hook]
/roteiro-leange                ← roteiro completo (motor: ogilvy-copy ou schwartz-copy)
    ↓
/legenda-para-reel
```

---

### Etapa 3 — Distribuição (opcional)

```
/1-conteudo-em-7-formatos   ← transforma o conteúdo aprovado em 7 formatos diferentes
    ↓
/publicar-social-leange       ← publica no Instagram, TikTok, LinkedIn
```

---

### Fluxo alternativo — fundo de funil

```
/banco-de-objecoes-do-avatar      ← mapeia objeções por ICP
    ↓ [escolhe objeção]
/carrossel-de-quebra-de-objecao   ← carrossel em 3 movimentos: nomeação → reframe → prova
    ↓
/carrossel-leange  +  /legenda-para-carrossel
```

---

**Aprovação humana obrigatória** em cada etapa — o fluxo para e aguarda antes de avançar.

---

## Regras do sistema

- Antes de executar qualquer tarefa, verificar se existe uma skill relevante em `.claude/skills/`
- Se encontrar, seguir as instruções da skill
- Conteúdo da empresa: sempre manter o contexto definido em `_contexto/empresa.md`
- Arquivos de credenciais: nunca commitar (estão no .gitignore)

---

## Fluxo de trabalho

Antes de executar qualquer tarefa, verificar se existe uma skill relevante em `.claude/skills/` ou `.claude/commands/`.
Se encontrar, seguir as instruções da skill.
Se não encontrar, executar a tarefa normalmente.

Ao concluir uma tarefa que não tinha skill mas parece repetível, perguntar:

> "Isso pode virar uma skill pra próxima vez. Quer que eu crie?"

Não perguntar pra tarefas pontuais ou perguntas simples. Só quando o padrão de repetição for claro.

---

## Aprender com correções

Quando o usuário corrigir algo ou dar instrução permanente ("na verdade é assim", "não faça mais isso", "sempre que...", "evita..."), perguntar:

> "Quer que eu salve isso pra não precisar repetir?"

Se sim, identificar onde salvar:

- **Sobre o negócio** → `_contexto/empresa.md`
- **Sobre preferências e estilo** → `_contexto/preferencias.md`
- **Sobre prioridades e foco atual** → `_contexto/estrategia.md`
- **Regra de comportamento nessa pasta** → `CLAUDE.md`

Salvar com uma linha nova clara, sem reformatar o arquivo inteiro.

---

## Criação de skills

Quando o usuário pedir pra criar uma nova skill:

1. Verificar se existe um template relevante em `templates/skills/`
2. Perguntar: "Essa skill é específica pra esse projeto ou vai ser útil em qualquer projeto?"
   - Específica desse negócio → `.claude/skills/nome-da-skill/SKILL.md` (local)
   - Útil em qualquer projeto → `~/.claude/skills/nome-da-skill/SKILL.md` (global)
3. Ler `_contexto/empresa.md` e `_contexto/preferencias.md` pra calibrar o conteúdo ao contexto
4. Se a skill precisar de arquivos de apoio, criar dentro da pasta da skill

---
> Source: [clara07004/design-LeAnge-serra](https://github.com/clara07004/design-LeAnge-serra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
