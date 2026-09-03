## cannonball

> Este repositório é um conjunto de **Agent Skills** mais o motor em Python que elas

# cannonball — para qualquer agente

Este repositório é um conjunto de **Agent Skills** mais o motor em Python que elas
chamam. Ele não depende de nenhum agente específico.

## O que você precisa saber para operar

**Cada pasta em `skills/` é autocontida.** Ela traz o `SKILL.md`, o `scripts/` que ele
chama, o `references/` que ele cita e o `seed/` das peças de exemplo. Instalar uma
skill é copiar a pasta dela; nada fica para trás.

**Resolva o caminho pelo arquivo, não por variável.** Todo comando nas skills usa
`${SKILL_DIR}`, que é a pasta do `SKILL.md` que você acabou de ler. Nenhuma variável
de ambiente de agente é necessária — e é isso que faz o mesmo `SKILL.md` funcionar em
Claude Code, Codex, Gemini CLI, Cursor e no resto.

**Requisito único: Python 3.** Os scripts importam só a biblioteca padrão. Sem
`pip install`, sem `node_modules`, sem rede — exceto onde o próprio comando diz que
usa rede (`curar.py --assets` testa URLs).

## Onde ficam os dados

O **acervo** é do usuário e mora fora deste repositório. A resolução, em ordem:

1. variável `CANNONBALL_ACERVO`
2. `cannonball.config.json` na raiz, se a pasta existir
3. `~/.cannonball/aonde` — ponteiro escrito por `scripts/vincular.py`
4. `../acervo` relativo aos scripts, se você estiver rodando de um clone com dados
5. `~/.cannonball` — padrão, criado na hora com as peças de exemplo

Nunca grave acervo dentro da pasta da skill instalada: em vários hospedeiros ela é
uma cópia em cache que a próxima atualização apaga.

## A ordem que dá melhor resultado

```
vincular.py --para <pasta>  →  kit-ingerir  →  kit-buscar  →  kit-cor + kit-tipo
   →  kit-montar  →  kit-otimizar-3d  →  kit-ingerir (guarda o que deu certo)
```

`python scripts/perfil.py` imprime essa ordem já adaptada ao estado do acervo.

## Ferramentas de agente citadas

As skills mencionam múltipla escolha para perguntas fechadas. Se o seu agente tiver
uma ferramenta dessas, use; se não, pergunte em texto — nada quebra.

Quatro MCPs são opcionais e cada skill só dispara a seção correspondente quando o
servidor está ligado: **GetLayers**, **Motion Sites**, **OriginKit** e **Higgsfield**.
Nenhum é requisito.

---
> Source: [harebeats/cannonball](https://github.com/harebeats/cannonball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
