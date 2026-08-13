## tc-2026-elemental-battle

> Projeto: **Elemental Battle** (TC-2026-ELEMENTAL-BATTLE)

# CLAUDE.md — Regras de Operação do Claude Code

Projeto: **Elemental Battle** (TC-2026-ELEMENTAL-BATTLE)
Disciplina: Tópicos em Computação · Python + Pygame
Rode o jogo com: `python -m meu_jogo.main`

Este arquivo é lido no início de toda sessão. Siga-o à risca.

---

## O QUE É O JOGO

RPG por turnos estilo Pokémon: overworld de exploração (mundo aberto com câmera que
segue o jogador) + batalhas por turno contra 4 chefes elementais. O jogador entra em
portais no campo de treino para lutar; vencer todos os 4 conclui o jogo.

Sistemas já implementados e **funcionando** (não quebrar):
- Áudio (`AudioManager`) — música e SFX gerados
- Pontuação (`ScoreSystem`) — combo, multiplicadores de tempo/HP, highscore
- Save (`SaveSystem`) — highscore em JSON
- Menu de batalha com 4 ações: **Atacar, Especial, Defender, Curar**
- IA Smart dos bosses
- Cenas: MenuScene, CampoDeTreinoScene, BattleScene, GameOverScene, VictoryScene
- Sistema de XP/nível em `character.py` (`gain_xp`, `level_up`)

---

## COMO EU TRABALHO COM VOCÊ (fluxo obrigatório)

Eu (Artur) desenho os prompts na chat do Claude e te entrego prompts em markdown com
**fases numeradas**. Sua execução segue estas regras sempre:

1. **Uma fase/tarefa por vez.** Ao terminar: resumo curto do que mudou + como testar.
   **PARE e aguarde meu OK** antes da fase seguinte. Nunca faça bulk de várias fases.
2. **Fase 0 de auditoria** (somente leitura) quando o prompt pedir: mapeie a estrutura
   real antes de editar qualquer coisa. Confirme comigo antes de tocar no código.
3. **Edições cirúrgicas com `str_replace`.** Nunca reescreva um arquivo inteiro se dá
   para editar um trecho. Não releia arquivos que já leu na mesma sessão.
4. **Respostas curtas.** Sem despejar arquivos inteiros na saída; mostre só os diffs
   relevantes e explique em poucas linhas.
5. Se um pedido conflitar com estas regras ou com a arquitetura, **pare e me pergunte**
   antes de improvisar.

---

## RESTRIÇÕES DA DISCIPLINA (valem nota — inegociáveis)

- **Orientação a objetos com herança/polimorfismo.** O professor já criticou
  fragmentação de lógica e "God-files"; arquitetura limpa é critério avaliado.
  Ao adicionar personagens, prefira uma hierarquia adequada (`Character` → `Player`/
  `Enemy`/`Boss`) a inchar uma classe só.
- **Sem novas dependências.** Apenas Python stdlib + Pygame (Box2D já está autorizado
  no projeto; qualquer outra lib precisa de aprovação do professor). Não instale nada.
- **Física correta com dt.** Posição += velocidade × dt (nunca posição += velocidade).
  Toda animação e movimento usam o `dt` do frame.
- **Vetores agrupados.** Use `pygame.Vector2` para posição/velocidade — nunca x e y
  soltos.
- **Ponto flutuante** para posição/velocidade; converta para pixel só ao renderizar.
- **Eu preciso saber explicar cada linha.** Gere código que eu entenda; comente as
  partes não óbvias. Nada de "mágica" que eu não consiga defender num questionamento.

---

## ARQUITETURA E ONDE CADA COISA VIVE

```
meu_jogo/
  main.py            → só inicializa GameManager + cena inicial. Sem lógica de jogo.
  core/              → motor: game_manager, scene_manager, game_state, config,
                       battle, game, map, elements  (+ progression, se criado)
  cenas/             → telas/renderização (menu, campo_de_treino, battle_scene, ...)
  entidades/         → character, acoes, ai  (estado + regras, não desenham na tela)
  data/              → characters_data, maps_data  (SÓ dados, sem lógica complexa)
  midia/sprites/     → sprite_factory (pixel art), animated_sprite (animações)
  utils/             → helpers (matemática, desenho, cores, debug)
  testes/            → protótipos e scripts; não fazem parte do jogo final
```

Regras de camada:
- `main.py` simples. Nada de batalha/XP/mapas aqui.
- `core/` controla funcionamento; **não** coloca sprites nem dados de inimigos aqui.
- `cenas/` desenham; **não** implementam regras de dano/XP.
- `entidades/` guardam estado e regras; **não** desenham direto na tela.
- `data/` só definições prontas.
- `midia/` só código de geração de arte/áudio; assets não viram lógica.

---

## CONVENÇÕES DE CÓDIGO

- **Comentários e mensagens em português.**
- **Constantes de tuning só em `core/config.py`.** Nada de números mágicos espalhados
  (tamanho da janela, FPS, XP por nível, incrementos por nível, multiplicadores de
  boss, etc.). Dados puros de personagem/mapa ficam em `data/`.
- **Reusar antes de reconstruir.** Antes de criar um sistema novo, verifique se já
  existe. Exemplos reais deste projeto:
  - XP/nível já existe em `character.py` → wire nele, não crie paralelo.
  - Cura (`HealAction`) já escala com `max_hp` → não duplicar essa lógica.
- **Sprites novos seguem o padrão da factory:** matriz 16×16, palette dict, caracteres
  de 1 letra, `'.'` = transparente. Não invente outro padrão — o professor espera
  consistência.
- **Tiles/sprites animados** usam `pygame.time.get_ticks()` para ciclos, ex:
  `phase = (pygame.time.get_ticks() % 1000) / 1000.0`.
- **Fallback preservado:** sprites/tiles antigos continuam funcionando se um novo
  falhar. Nada de remover o caminho antigo sem substituto testado.

---

## TRABALHO ATIVO: BUG DA MORTE + PROGRESSÃO + ESCALONAMENTO DE BOSSES

Workstream em andamento, guiado pelo arquivo `PROMPT_PROGRESSAO_CLAUDE_CODE.md` (5
fases com gate de confirmação cada uma: Auditoria → Corrigir travamento na morte →
Progressão do herói → Escalonamento dos bosses → Balanceamento/validação). Quando eu
pedir para seguir com esse workstream, use aquele prompt como roteiro de fases — as
regras abaixo são o contexto permanente que sustenta ele.

**Objetivo do workstream:**
1. Corrigir o travamento quando o jogador morre (derrota não chega ao GameOverScene).
2. Herói ganha +1 nível por boss derrotado (vida, ataque, defesa e cura sobem).
3. O primeiro boss enfrentado é sempre o mais fácil, não importa qual portal o
   jogador escolher primeiro; cada boss seguinte sobe de nível e fica mais difícil.
4. Balancear para o herói ficar mais forte sem virar overpower em relação ao boss.

**Fatos já mapeados no código (não redescobrir — só confirmar na Fase 0):**
- Sistema de XP/nível **já existe** em `entidades/character.py` (`level`, `xp`,
  `gain_xp()`, `level_up()`), usando `XP_PER_LEVEL`, `HP_LEVEL_INCREMENT`,
  `DAMAGE_LEVEL_INCREMENT`, `DEFENSE_LEVEL_INCREMENT` de `core/config.py`.
  `level_up()` já restaura HP cheio e dispara `on_level_up` (registrado em `main.py`
  com notificação + SFX). **Reusar — não criar sistema paralelo.**
- `entidades/acoes.py`: Especial = dano ×1.8 (3 usos), Defender = 50% de redução
  (2 usos), Curar = 30% do `max_hp` (2 usos) — a cura **já escala sozinha** quando
  `max_hp` cresce por nível.
- `data/characters_data.py`: bosses hoje são **singletons de módulo** com dificuldade
  fixa por elemento (Hydra 100/16/4 → Magma Titan 160/28/8) + helper `reset_boss()`.
  Isso conflita com o mundo aberto: escolher o portal do Magma Titan primeiro dá de
  cara com o boss mais forte.
- Herói base em `main.py`: 120 HP / 20 dano / 5 defesa, elemento Fire, fraqueza Water.
- `cenas/battle_scene.py`: bloco "animação de morte antes de finalizar batalha"
  (`_death_started`, `finished`) — provável origem do travamento na morte do jogador.
- `core/game.py` mantém `defeated_bosses: set` — fonte de verdade da progressão.
- `ScoreSystem.finalizar_batalha()` só é chamado em vitória e usa `enemy.is_boss`.

**Regras específicas deste workstream:**
- Progressão aplicada na **criação/preparação dos personagens**, nunca dentro do
  fluxo de turnos (`core/battle.py`/`BattleScene` — turn-flow será refatorado à
  parte; não engordar essa área agora).
- Progressão é **por partida** (runtime); não alterar o formato do save.
- Arquivos novos permitidos neste workstream: `core/progression.py` e
  `meu_jogo/testes/balance_report.py`. Nada além disso sem combinar.

---

## ARMADILHAS CONHECIDAS DESTE PROJETO

- **Bosses como singletons de módulo** (`data/characters_data.py`) têm stats fixos por
  elemento e mutáveis — vazam estado entre lutas e quebram o escalonamento por ordem
  de enfrentamento. Quando o design pedir dificuldade progressiva, crie **instância
  nova por batalha**, nunca reutilize o singleton.
- **`Character.take_damage` não faz clamp** — `self.hp -= real_damage` pode deixar HP
  negativo. Ao mexer em dano/morte, garanta `hp = max(hp - real_damage, 0)` mantendo
  o retorno de `real_damage` inalterado (o ScoreSystem depende dele).
- **Fluxo de morte na BattleScene** pode travar se o personagem não tiver animação de
  morte registrada (espera infinita por `is_finished`). Bloqueios de animação
  precisam de timeout de segurança (~1.5–2.0s).
- **`defeated_bosses` (set) em `core/game.py` é a fonte de verdade** da progressão do
  mundo aberto. Nível do herói e nível do boss devem derivar dela — não crie
  contadores paralelos.
- **`ScoreSystem` depende de `enemy.is_boss`** e do valor de retorno de `take_damage`.
  Não altere essas assinaturas sem avisar.

---

## NÃO QUEBRAR

- AudioManager, ScoreSystem, SaveSystem, Smart AI.
- As 4 ações do menu de batalha.
- Fluxo Menu → Overworld → Batalha → GameOver/Victory.
- Formato do JSON do save (progressão de nível é por partida/runtime; não persista
  no save sem me pedir).

---

## GIT / ENTREGA

- Commits pequenos e frequentes (mínimo 1 por semana), mensagens claras em português
  descrevendo a mudança real. Nada de commit "falso".
- Ao concluir uma fase aprovada, sugira uma mensagem de commit; **eu** faço o commit.

---

## RESUMO DE UMA LINHA

Faça uma fase por vez, edite cirurgicamente, mantenha OO limpo e dt-based, centralize
tuning no config, reuse o que já existe, e pare para eu confirmar antes de seguir.

---
> Source: [Game-4ano/TC-2026-Elemental-Battle](https://github.com/Game-4ano/TC-2026-Elemental-Battle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
