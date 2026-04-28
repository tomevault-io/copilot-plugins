## adaptacorestudio

> Antes de qualquer ação, leia e aplique a skill em:

# PROTOCOLO OBRIGATÓRIO — ATIVO EM CADA MENSAGEM

Antes de qualquer ação, leia e aplique a skill em:
`.agents/skills/interpretacao_de_comandos/SKILL.md`

Resumo do protocolo:
1. Liste APENAS o que foi pedido explicitamente (Passo 1 da skill)
2. Classifique o tipo de pedido (Passo 2)
3. Verifique o escopo — "simples" = mínimo, alvo específico = só aquele arquivo (Passo 3)
4. Nada que não foi pedido é executado. Extras são sugeridos, nunca executados (Passo 4)
5. Em caso de dúvida, pergunte antes de agir (Passo 5)

---


A Gabriela é a sua usuária/Tech Lead.

# Diretriz Inviolável: Cores e Design System
O CoreStudio possui um Design System estabelecido através de variáveis CSS (ex: em `App.css` e `.module.css` files) e sua Cartilha Cromática de Marca.

1. **NUNCA INVENTE CORES**: É totalmente proibido adicionar novas cores via Hexadecimal (ex: `#FF0000`), HSL (ex: `hsl(210, 50%, 50%)`), RGB/RGBA, ou utilitários CSS arbitrários que não façam parte do tema (ex: `text-red-500` alienígenas ao contexto abstrato do tema).
2. **USE VARIÁVEIS**: Sempre busque pelas variáveis CSS já existentes no projeto para fundos, fontes e bordas (ex: `var(--bg-primary)`, `var(--text-secondary)`, `var(--border)`, `var(--primary)`).
3. **PEÇA PERMISSÃO**: Se ao criar um novo elemento visual você achar estritamente necessário utilizar uma cor nova que ainda não parece constar no sistema, pare. Redija uma menção a Gabriela explicando por que precisa de uma cor ali e pergunte qual cor do brand/design system você deve aplicar.

O descumprimento desta regra quebra as diretrizes de harmonia visual glassmorphism/premium da Adapta.

---
> Source: [gabriela92dias-web/adaptacorestudio](https://github.com/gabriela92dias-web/adaptacorestudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
