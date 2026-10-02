## escritonasestrelas

> Site pessoal de página única. Conta o tempo até minha namorada voltar da Dinamarca.

# Contador

Site pessoal de página única. Conta o tempo até minha namorada voltar da Dinamarca.

`reference.html`, na raiz, é a especificação visual: paleta, tipografia,
geometria do arco, matemática das caudas e textos saem de lá. Onde este
arquivo divergir do reference, vale este arquivo.

## Dados fixos
- Rio de Janeiro: -22.9068, -43.1729
- Vejle, Dinamarca: 55.7093, 9.5360
- Ida (início da contagem): 2026-09-01T12:00:00-03:00
- Volta (alvo): 2027-05-31T12:00:00-03:00
- Distância Rio–Vejle: constante, ~10.053 km (haversine, R=6371). Nunca
  mostrar distância que diminui com o tempo.
- A data da volta na tela sempre sai de `Intl.DateTimeFormat` a partir da data
  de volta. Nunca escrever a data à mão, nem copiar a do reference.

## Conceito central
Progresso p = (agora - ida) / (volta - ida), limitado entre 0 e 1.
Toda a interface é função de p. Em p=0 estamos nos extremos; em p=1 nos encontramos.

## Regras técnicas
- Vite + React + TypeScript. Sem backend, sem banco, sem autenticação. Build estático.
- Animação com `requestAnimationFrame`, sempre cancelada no cleanup do efeito.
  Sem Framer Motion.
- A cena anima de 0 até o progresso atual a cada montagem. Não existe animar
  "a partir da última visita": o delta entre visitas vive só no texto, a partir
  da chave `lastVisit`.
- Em p >= 1 a página vira o estado de chegada (`Arrival`), que incorpora o clarão.
- Gráficos em SVG inline. Nenhuma imagem remota; assets locais são permitidos.
  A foto fica em `public/nos.jpg` e é referenciada como `/nos.jpg`, nunca data URI.
- Fontes: só Google Fonts.
- Mobile first: o uso principal é celular.
- Sem dependências novas sem me perguntar antes.

## Estrutura
- `src/time.ts`, `src/geo.ts` e `src/storage.ts` ficam onde estão. Não criar `src/lib/`.
- Tokens visuais em `src/styles/tokens.css`; reset mínimo em `src/index.css`.
- O lab de teste fica atrás de `#lab`, fora da página pública. A seção de
  paleta e tipografia do reference não faz parte do site.

## Identidade visual
Instrument Serif (400, com itálico) para números e ênfase; Inter (200/300/400)
para rótulos e texto. Instrument Serif não tem peso alto: **a hierarquia vem de
tamanho e cor, nunca de peso**.

Sem simbologia romântica (corações, rosas, fitas) e sem vermelho saturado como
cor de marca.

## Tokens
Os nomes são os do projeto; os valores vêm do reference.

### Cor
| Token | Valor | No reference | Papel |
|---|---|---|---|
| `--color-night-900` | `#05070F` | `--night` | fundo; base do céu |
| `--color-night-800` | `#080E22` | céu 26% | |
| `--color-night-700` | `#141839` | céu 52% | |
| `--color-night-600` | `#241E4C` | `--violet`, céu 74% | |
| `--color-night-500` | `#332249` | céu 100% | |
| `--color-coral` | `#E8734F` | `--coral` | brilho do horizonte |
| `--color-ember` | `#FF9D5C` | `--ember` | cometa do Rio (eu) |
| `--color-frost` | `#86D8E8` | `--frost` | cometa de Vejle (ela) |
| `--color-gold` | `#FFC978` | `--gold` | clarão do encontro; ênfase no delta |
| `--color-ember-core` | `#FFD9AE` | núcleo da cabeça A | |
| `--color-ember-tail` | `#FFB877` | cauda A | |
| `--color-frost-core` | `#DFF6FA` | núcleo da cabeça B | |
| `--color-frost-tail` | `#A9E6F2` | cauda B | |
| `--color-text-strong` | `#F6F2EA` | `--text` | |
| `--color-text-muted` | `#9298AE` | `--muted` | |
| `--color-text-faint` | `#868DA5` | `--dim` (clareado) | o `#565D78` do reference dava 3,1:1 sobre o fundo; este dá 6,1:1 |
| `--color-line` | `rgba(246,242,234,.10)` | `--line` | bordas |
| `--gradient-night` | 2 radiais + 1 linear | `.sky` | céu |

### Tipografia
| Token | Valor | No reference |
|---|---|---|
| `--text-xs` | `.52rem` | `.lbl` |
| `--text-sm` | `.56rem` | `.clock .n` |
| `--text-base` | `.64rem` | `.eyebrow`, `.cities` |
| `--text-lg` | `.8rem` | `.delta` |
| `--text-xl` | `1.7rem` | `.clock .t` |
| `--text-2xl` | `clamp(1.3rem,5.2vw,1.95rem)` | `.opening` |
| `--text-3xl` | `clamp(1.6rem,7vw,2.3rem)` | `.dist .date` |
| `--text-4xl` | `clamp(1.8rem,9vw,3.4rem)` | `.num` |

`--weight-light` 300 (peso do corpo), `--weight-regular` 400. O reference carrega
Inter 200 mas não usa em lugar nenhum; não carregamos.

`--photo-op` é a opacidade da foto atrás da cena. Não é token global: a cena
escreve no wrapper a cada quadro, a partir do mesmo valor animado que move os
cometas, entre `PHOTO_OP_MIN` e `PHOTO_OP_MAX` (constantes em `Scene.tsx`).
`--leading-tight` .95, `--leading-snug` 1.1, `--leading-normal` 1.34.
`--tracking-label` .14em, `--tracking-wide` .18em, `--tracking-eyebrow` .24em.

### Espaço e raio
`--space-1` .5rem, `--space-2` 1rem, `--space-3` 1.5rem, `--space-4` 2.5rem,
`--space-5` 4rem, `--space-6` 6rem (os `--s1`..`--s6` do reference).
`--radius-md` 14px (`--r`), `--radius-full` 999px.

Espaçamentos menores que `--space-1` seguem o reference literalmente (.25rem no
vão da contagem, .6rem sob os números).

## Estilo de trabalho
- Uma tarefa por vez. Não antecipe features que eu não pedi.
- Prefira poucos arquivos bem nomeados a muitas abstrações.

---
> Source: [alobogiron/escritoNasEstrelas](https://github.com/alobogiron/escritoNasEstrelas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
