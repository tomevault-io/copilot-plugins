## cojeprim

> Fonte técnica da marca: `src/brand.ts` (cores, fontes, logo, formato).

# Cojeprim — Guia de Identidade Visual & Motion

Fonte técnica da marca: `src/brand.ts` (cores, fontes, logo, formato).
Nunca hardcode cor/fonte/logo numa composição — importe sempre de `src/brand.ts`.

## Essência
Identidade jovem, chique e vibrante — instituição para jovens empreendedores.
Visual criativo, movimentado e cheio de energia, sem nunca perder organização e alinhamento.
Energia + sofisticação ao mesmo tempo: jovem, mas nunca bagunçada.

## Cores
- Fundo padrão: branco `#FFFFFF`.
- Elementos gráficos: verde `#009345`.
- Texto: verde `#009345` sobre branco, ou branco sobre verde.
- Cor de destaque: usar uma terceira cor APENAS para info importante/indicadores. Nunca mais de uma cor de destaque por vez.

## Formas
- Quadradas/retangulares com cantos arredondados.
- Círculos como elemento de dinamismo.
- Bordas de 2px (nunca grossas).
- Raios de borda consistentes entre todos os elementos.
- Nunca sombras exageradas; evite efeitos estáticos chamativos (o destaque é o motion).

## Hierarquia
- Espaço em branco generoso, mesmo em telas dinâmicas.
- Poucos elementos por tela; um único ponto focal por vez.
- Margens consistentes, alinhamentos precisos a um grid invisível.
- Na dúvida entre adicionar ou remover: remova. Menos gera mais impacto.

## Tipografia
- Rockwell Bold para destaque; Skia Regular para apoio (editorial + jovem).
- Pesos diferentes para hierarquia; evite excesso de negrito.
- Bastante espaçamento entre blocos; nunca comprima textos; poucas palavras por tela.

## Motion (assinatura da marca)
Priorize:
- Motion blur em transições e movimentos rápidos.
- Movimentos dinâmicos com ritmo/velocidade perceptíveis.
- Ease In Out nas entradas e saídas, mesmo em movimentos rápidos.
- Curvas fluidas; ritmo consistente entre cenas; cortes energéticos mas organizados.

Nunca:
- Bounce exagerado, overshoot forte, elastic.
- Tremores sem propósito; movimentos que quebrem o grid; mudanças de velocidade que pareçam erro técnico.

### Entrada de elementos
- Fade com motion blur; deslocamento vertical/horizontal com velocidade.
- Scale com leve overshoot controlado (nunca elástico).
- Entrar na direção do movimento da composição. Motion não deve ser escondido.

### Saída de elementos
- Fade com motion blur; deslocamento rápido e direcional; scale down controlado.
- Nunca sumir de forma seca/sem transição.

### Velocidade
- Energia jovem + premium. Evite motion lento demais, caos sem alinhamento, movimentos sem direção.
- Rápido quando fizer sentido, mas sempre limpo e intencional.

## Organização & Espaçamento
- Tudo alinhado a grids invisíveis, mesmo em movimento (textos, ícones, botões, cards, linhas).
- Nada "solto", nem durante a animação.
- Espaçamentos generosos; leveza mesmo sendo dinâmico.

## Componentes
- Linhas: 2px, retas, precisas, cantos arredondados quando necessário, motion blur quando em movimento.
- Ícones: minimalistas, outline, traço fino, geometria simples (quadrados/círculos), sem excesso de detalhes.
- Cards: fundo branco ou verde `#009345`, borda 2px na cor de contraste, cantos arredondados, bom espaço interno, pouca info por card.

## Composição
- Organização mesmo sendo vibrante. Evite poluição visual, muitos elementos competindo, muitas cores, muitos tamanhos de fonte.
- Máximo de cores: verde + branco + uma cor de destaque.
- Cada tela tem um objetivo.

## Formato
- Vertical 1080x1920 (9:16), 30fps — padrão Cojeprim (`videoFormats.vertical`).

## Filosofia
Energia · Precisão · Clareza · Sofisticação jovem · Movimento com propósito · Espaço em branco · Consistência · Legibilidade · Harmonia chique e vibrante.

---
> Source: [arcocontents-dotcom/cojeprim](https://github.com/arcocontents-dotcom/cojeprim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
