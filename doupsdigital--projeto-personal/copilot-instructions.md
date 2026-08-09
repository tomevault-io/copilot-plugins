## projeto-personal

> Este é o projeto-base usado para gerar páginas premium de apresentação

# Projeto: Páginas Premium de Portfólio (Base)

Este é o projeto-base usado para gerar páginas premium de apresentação
pessoal para profissionais autônomos (nutricionistas, personal trainers,
modelos comerciais, tatuadores, influencers, etc). Este arquivo define as
convenções que devem ser seguidas em TODA página gerada a partir deste
template.

Este repositório específico também é o laboratório onde o template foi
construído e testado (cliente fictícia "Isabella Marques", 5 rotas de
demonstração — ver "Direções de partida" abaixo). Pra um cliente novo,
ele é **duplicado** — ver "Fluxo de trabalho por cliente".

## O que este projeto NÃO é

Isto NÃO é uma landing page de vendas. Não existe produto, checkout,
oferta ou "comprar agora". A pessoa é o produto — a página existe para
apresentá-la da forma mais impactante possível e gerar em quem visita o
desejo de contratá-la, agendar com ela ou fechar uma parceria.

Pense nela como um "link na bio" premium: substitui o Linktree/WhatsApp
genérico por uma experiência editorial que reforça autoridade e desejo
antes mesmo da pessoa conversar com o cliente/parceiro.

## Stack técnica

- React + Vite + TypeScript
- Tailwind CSS para estilização
- GSAP + ScrollTrigger para animação (ver "O motor comprovado" e
  "Experiência de scroll" — como exatamente cada mecanismo é usado)
- Framer Motion apenas para transições simples de UI que NÃO dependem
  da posição do scroll (hover de botão, abrir/fechar menu, troca de
  rota)
- Antes de escrever um componente do zero, **verificar se já existe um
  equivalente em `src/components/`** (ver "O motor comprovado" abaixo) —
  esse é o ativo mais valioso deste projeto, já testado em 5 paletas
  diferentes.
- Deploy via Vercel

## Estrutura de pastas

```
src/
  components/     # componentes reutilizáveis e já comprovados (Hero, Sobre,
                   # Portfólio, Campanhas, Processo, Contato, etc.) — ver
                   # "O motor comprovado" abaixo
  pages/          # uma página por rota; durante a fase de propostas tem
                   # várias (ver "Múltiplas propostas por cliente"), mas o
                   # projeto final de um cliente é normalmente uma só
  content/        # dados do cliente atual (bio, stats, portfólio, links)
                   # num único arquivo tipado — renomear por cliente
  assets/         # imagens, vídeos, ícones já processados — organizados em
                   # portrait/ e landscape/ dentro de images/ e videos/ (ver
                   # "Regras de design")
  lib/            # helpers (setup do gsap, conversão de cor, etc.)
Context/          # (não versionado) prints, bio, fotos/vídeos originais e
                   # brief do cliente atual — ver .gitignore
```

## O motor comprovado: reaproveitar, não recriar

Este projeto passou por bastante iteração até chegar num resultado que
funciona bem (várias abordagens de transição entre seções foram tentadas
e descartadas antes desta, ver o "por quê" logo abaixo). O resultado está
validado em produção nas rotas `/vinho`,
`/noir`, `/riviera`, `/studio` e `/cover`, cada uma com paleta e
tipografia próprias sobre a mesma estrutura. **Não reinvente essas peças
pra um cliente novo** — componha a partir delas, e só escreva um
componente novo quando a seção realmente não tiver equivalente.

### Componentes prontos (`src/components/`)

- `VideoHero.tsx` — abertura em vídeo full-bleed, textos entrando com
  GSAP, `replayOnScroll` pra reanimar a entrada toda vez que a seção volta
  a ficar visível.
- `AboutSection.tsx` — foto de fundo com parallax sutil, bio + stats
  embaixo, "eyebrow" fixo (`position: sticky`) no topo da seção.
- `PinnedPortfolio.tsx` — a galeria principal: cada item é uma seção
  cheia, troca por fade de opacidade com duração fixa (não escrubado) ao
  ficar ativa. É a seção mais importante da página (ver "Estrutura de
  conteúdo").
- `CampaignsSection.tsx` — chips de categoria/especialidade com ícone
  opcional (`CategoryIcon.tsx`) + carrossel de marcas parceiras opcional
  (`BrandsMarquee.tsx`).
- `ProcessSection.tsx` — linha do tempo "como funciona" (numerada, com
  foto de fundo escurecida). Genérica o suficiente pra qualquer nicho com
  um processo de trabalho (briefing → produção → entrega; avaliação →
  plano → acompanhamento; orçamento → sessão → entrega de fotos) — só
  trocar os textos dos passos.
- `ContactSection.tsx` — fechamento com foto de fundo, CTA principal,
  bloco opcional de material de apoio (mídia kit, tabela de valores,
  portfólio em PDF), barra fixa com Instagram/e-mail/localização.
- Suporte: `FullBleedMedia.tsx` (crop 9:16/16:9 sem esticar/cortar mal;
  aceita `desktopSrc` pra trocar a mídia por uma versão landscape só no
  desktop — ver "Regras de design" — via JS `matchMedia`, nunca `<source
  media>` dentro de `<video>`, que não tem suporte confiável entre
  browsers), `MagneticGlowButton.tsx` e `CTAButton.tsx` (CTAs),
  `ScrollFade.tsx` (reveal simples pra conteúdo fora do container de
  scroll-snap), `CategoryIcon.tsx`, `BrandsMarquee.tsx`.

Todo componente recebe cor/fonte via props — nunca tem cor hardcoded
internamente. Adaptar o visual pra um cliente novo é passar props
diferentes (ver "Direções de partida"), não editar o componente.

### Contrato de comportamento (aplicar sempre, em toda seção)

- **Um único container de scroll por página**, com `id` próprio e
  `h-dvh w-full snap-y snap-mandatory overflow-y-auto` — nunca
  `min-h-screen` solto nem um sub-scroller aninhado dentro da página.
  Todas as seções principais (Hero, Sobre, cada item do Portfólio,
  Campanhas, Processo, Contato) são `snap-start snap-always` desse mesmo
  scroller, do início ao fim — uma rolagem sempre avança exatamente uma
  seção, sem ponto onde a física do scroll muda.
  - **Por que CSS scroll-snap nativo, e não GSAP `scrub`/`pin` entre
    seções**: já foi tentado e descartado — `ScrollTrigger.pin` com
    scroller próprio travava e gerava blocos pretos no celular, e um
    crossfade escrubado 1:1 com a distância de scroll ficava instável em
    rolagem rápida (fling). O scroll-snap nativo do navegador resolve
    isso de forma muito mais robusta.
- **Detectar a seção "ativa" com `IntersectionObserver`** (threshold
  ~0.6, constante `ACTIVE_THRESHOLD` no topo de cada componente que
  precisa disso), nunca por cálculo de posição de scroll.
- **Reveal de texto/conteúdo com duração fixa via CSS**
  (`transition-[opacity,transform] duration-700` a `duration-1000`)
  quando o estado ativo muda — não escrubado 1:1 com o scroll. GSAP com
  `scrub` continua valendo só pra efeitos **dentro** de uma seção já
  ativa (parallax leve de imagem de fundo, ken-burns/zoom contínuo) —
  nunca pra fazer a transição inteira entre duas seções.
- **Label fixo ("eyebrow") por seção**: `position: sticky` no topo da
  seção, sempre na cor `accent` da paleta (salvo exceção documentada —
  ex: um `accent` escuro que não teria contraste sobre uma seção com foto
  bem escura, aí usa-se um tom mais claro só ali).
- **Hero (e qualquer seção com animação de entrada)**: usar o padrão
  `replayOnScroll` — a timeline do GSAP fica pausada e reinicia via
  `IntersectionObserver` toda vez que a seção volta a ficar visível, não
  só uma vez no load.
- **Escurecer mídia de fundo pra dar contraste ao texto**:
  `imageBrightness`/`mediaBrightness` em torno de `0.4` nas seções com
  foto grande atrás de texto (Sobre, Portfólio) — ajustar caso a caso se
  o texto não ficar legível o suficiente.
- **Seções com pouco conteúdo dentro do modelo `h-dvh`/scroll-snap podem
  sobrar bastante espaço vertical vazio** — conferir em tela real (não só
  no preview) se isso incomoda antes de considerar a seção fechada.
- **Testar o scroll-snap tanto por toque/swipe (mobile) quanto por roda
  do mouse/trackpad (desktop)** — o comportamento de fling/momentum não é
  idêntico entre os dois, e boa parte da validação deste padrão foi feita
  só no celular.
- Este contrato **substitui** a ideia de "scroll totalmente escrubado
  estilo Apple" que existia numa versão anterior deste arquivo — ver
  "Experiência de scroll" mais abaixo pro texto atualizado.

### Direções de partida (paletas de referência)

As rotas `/noir`, `/riviera`, `/studio` e `/cover` (além da `/vinho`, a
mais trabalhada de todas) ficam neste projeto-base como **pontos de
partida rápidos de humor visual** — não são "os 5 estilos definitivos pra
qualquer cliente": foram criadas comparando direções pra uma única
cliente fictícia. Use como inspiração de arranjo (paleta escura vs. clara,
serifada vs. mono, editorial vs. dramática) e depois **redirecione
cores/tipografia pra identidade real do cliente atual** — não copie os
hex literalmente a menos que já combinem por coincidência com a marca
dele.

- **Vinho** (`/vinho`) — marrom quase preto + dourado + vinho, Playfair
  Display. A página mais completa e testada — comece por ela como
  referência de estrutura ao montar um projeto novo.
- **Editorial Noir** (`/noir`) — preto + dourado, Fraunces itálico —
  editorial/autoral.
- **Riviera Gold** (`/riviera`) — areia + bronze, Cormorant Garamond —
  quente/refinado.
- **Studio Clean** (`/studio`) — branco + verde-sálvia, Space
  Grotesk/mono — clean/orientado a dados.
- **Bold Cover** (`/cover`) — verde-oliva escuro + dourado, DM Serif
  gigante — dramático, estilo capa de revista.

### Convenções de CTA e copy padrão comprovadas

O padrão abaixo (a **estrutura**, não o texto literal) já foi validado
nas 5 rotas — usar como forma, adaptando as palavras à voz e ao nicho do
cliente real:

- Hero: um único CTA principal (sem secundário por padrão — ver "Regras
  de CTA"), com uma legenda curta opcional na fonte de destaque e cor
  `accent` logo acima do botão (ex: "Collabs e Campanhas" pra um perfil
  de modelo; pra outro nicho seria algo como "Consultas Abertas" ou
  "Agenda 2026").
- Contato final: título de impacto + o mesmo CTA principal + bloco
  opcional de material de apoio (mídia kit, tabela de valores, portfólio
  em PDF) abaixo de um divisor fino — só incluir se fizer sentido pro
  nicho.
- Isso é a implementação visual da regra de "um único CTA, nunca
  insistente" já descrita em "Regras de CTA" — não uma regra nova.

## Múltiplas propostas por cliente (liberdade criativa)

A estrutura da vinho (ordem de seções, comportamento de scroll, elementos
de assinatura) foi validada a fundo, mas só pro nicho de modelo comercial.
Pra qualquer cliente novo — de qualquer nicho — **gerar no mínimo 2
páginas, idealmente 3 a 4**, antes de fechar num resultado único:

1. **Uma página "base"** — segue à risca a estrutura, ordem de seções e
   comportamento de scroll já comprovados (ver "O motor comprovado"), só
   adaptando conteúdo/paleta/tipografia ao cliente. É a opção segura, já
   testada de ponta a ponta.
2. **Mais 2 ou 3 páginas com liberdade criativa de verdade** — não são só
   recolorações da mesma página (isso já foi feito nas 5 direções da
   Isabella e não é o objetivo aqui). São propostas genuinamente
   diferentes de estrutura visual, ordem/ênfase das seções e elementos de
   assinatura (como a régua de alfaiataria da Riviera ou o Comp Card da
   Studio foram tentativas próprias, cada uma) — usando o julgamento
   criativo do Claude Code, sempre dentro das regras de nicho/tom/design
   já documentadas neste arquivo (ver "O que este projeto NÃO é", "Tom de
   copy por nicho", "Regras de design").
3. **O motor técnico de scroll não entra na liberdade criativa** — toda
   proposta, mesmo as mais ousadas, usa o mesmo contrato de comportamento
   comprovado (scroll-snap + `IntersectionObserver` + GSAP só pra efeitos
   internos, ver "O motor comprovado"). A liberdade é sobre
   estrutura/conteúdo/visual, não sobre reabrir decisões técnicas de
   scroll que já custaram várias iterações pra acertar.
4. O usuário revisa todas as propostas (cada uma numa rota própria, ex:
   `/base`, `/proposta-2`, `/proposta-3`) e decide qual seguir — ou pede
   pra misturar elementos de mais de uma. A palavra final é sempre dele;
   as propostas são ponto de partida pra decisão, não a entrega em si.
5. Só depois de escolhida a direção (ou combinação de direções), colapsar
   pro projeto final de página única — ver "Fluxo de trabalho por
   cliente" abaixo.

Isso é diferente do fluxo de `Context/Referencias/` (ver "Referências
visuais"): lá o cliente já forneceu referências visuais prontas pra
seguir; aqui é o Claude Code propondo variações próprias quando não há
essas referências — os dois fluxos podem se combinar (usar as referências
do cliente como uma das propostas, e explorar mais 1 ou 2 direções
próprias além delas).

## Fluxo de trabalho por cliente

1. Duplicar este repositório (GitHub) inteiro pra um novo, renomeado com
   o nome/nicho do cliente.
2. Abrir uma sessão nova do Claude Code nesse repositório e pedir pra ler
   este `CLAUDE.md`.
3. Fornecer o que tiver do cliente — dois cenários possíveis, ambos
   válidos:
   - **Com assets**: colocar em `Context/` prints, bio, fotos/vídeos de
     trabalho, depoimentos, paleta de cores da marca pessoal e links de
     contato — e pedir pra montar a página com base nisso.
   - **Só a ideia**: descrever o nicho, o nome/marca, o tom desejado e
     (se quiser) uma paleta — sem fotos/vídeos/bio prontos. Ver "Modo sem
     assets" abaixo.
4. Escolher (ou pedir sugestão de) uma das "Direções de partida" acima
   como ponto de partida de humor visual pra página base, e então adaptar
   a paleta/tipografia pra identidade real do cliente.
5. Montar **no mínimo 2 propostas, idealmente 3 a 4** (ver "Múltiplas
   propostas por cliente" acima), reaproveitando os componentes de
   `src/components/` (ver "O motor comprovado") em todas elas, seguindo a
   ordem de seções e as regras de tom/design/scroll deste arquivo — a
   liberdade criativa das propostas extras é de estrutura/visual, não de
   mecânica de scroll.
6. O cliente revisa as propostas e escolhe a direção (ou combinação de
   direções) — só então fechar o resultado.
7. Colapsar pro projeto final: **uma página única em `/`** — não deixar a
   Home seletora de direções nem as rotas das propostas descartadas no
   projeto entregue. Isso inclui remover o link de seta (←) fixo no canto
   superior esquerdo dos Heros — é resquício de navegação entre rotas de
   comparação e não tem função numa página única.
8. Revisar manualmente espaçamento, contraste, qualidade de imagem/vídeo
   e timing das animações antes de publicar.
9. Rodar `vercel --prod` pra publicar.

## Modo sem assets (só a ideia do projeto)

Quando o usuário passar só a ideia/briefing do cliente, sem fotos, vídeos
ou bio prontos:

- **Onde entraria foto/vídeo** (Hero, Sobre, Portfólio): usar um bloco de
  cor sólida ou gradiente (derivado da paleta escolhida pro nicho) com o
  nome/iniciais do cliente centralizado, no lugar do `src` real — mantém
  a animação de scroll funcionando normalmente (parallax, escurecimento,
  reveal), só sem mídia de verdade. Nunca inventar/usar URL de banco de
  imagens externo.
- **Copy**: pode escrever bio, legendas de portfólio e categorias
  seguindo o tom do nicho (ver "Tom de copy por nicho"), mas **avisar
  claramente ao usuário quais textos foram inventados** — ele decide
  deixar isso a cargo do Claude Code quando não fornece copy própria, mas
  o texto final ainda precisa da revisão dele antes de publicar.
- Sinalizar no início da conversa (ou nas primeiras respostas) que aquele
  projeto está em modo placeholder, pra não ser confundido com um projeto
  pronto pra publicar.

## Referências visuais (pasta `Context/Referencias/`)

Esse fluxo é complementar ao de "Múltiplas propostas por cliente": ali é
o Claude Code propondo variações próprias; aqui é o cliente que já
forneceu referências visuais prontas pra seguir. Os dois podem se
combinar — usar as referências do cliente como uma (ou mais) das
propostas, e ainda explorar 1-2 direções próprias além delas.

Quando `Context/Referencias/` existir, ela contém uma ou mais direções
visuais de referência (arquivos HTML leves, só com cores, tipografia e
estrutura de layout — sem fotos reais e sem o conteúdo final). Cada
arquivo é uma direção de estilo diferente para a mesma pessoa/cliente.

Ao encontrar essa pasta:

- Ler cada arquivo de referência antes de gerar qualquer código —
  extrair paleta de cores exata (hex), tipografia (nome das fontes de
  título/corpo/dados) e o elemento de assinatura descrito em cada um.
- Gerar **uma página completa por referência**, em rotas separadas
  dentro do mesmo projeto (ex: `/noir`, `/riviera`, `/studio`,
  `/cover`, `/vinho` — usar um nome curto baseado no nome do arquivo de
  referência para cada rota).
- Cada rota deve usar o conteúdo real do cliente (fotos e vídeos de
  `Context/Imagens/` e `Context/Videos/`, texto de `Context/Bio/`),
  seguindo fielmente a paleta e a tipografia daquela referência
  específica — nunca misturar cores/fontes de uma referência com o
  layout de outra.
- Manter a mesma estrutura de conteúdo (ver seção "Estrutura de
  conteúdo" abaixo) em todas as rotas, variando apenas a personalidade
  visual — isso facilita comparar as opções lado a lado.
- Se não houver pasta `Referencias/`, seguir as regras de design gerais
  deste arquivo e o fluxo de "Múltiplas propostas por cliente" (gerar
  mais de uma direção, derivadas da identidade do próprio cliente — não
  parar numa única direção de cara).

## Estrutura de conteúdo (ordem das seções)

Diferente de uma landing de vendas, o fluxo aqui é apresentação →
prova de trabalho → desejo → contato. Ordem padrão, salvo indicação
contrária:

1. **Abertura de impacto** — vídeo em loop de fundo (ver "Experiência
   de scroll"), nome/marca pessoal e frase de posicionamento
   sobrepostos na frente do vídeo. Nunca uma foto estática sozinha
   como fundo do hero quando houver vídeo disponível.
2. **Sobre / trajetória** — quem é a pessoa, o que ela representa,
   credenciais ou experiência relevantes, em tom de storytelling, não
   de currículo.
3. **Portfólio / trabalho** — a seção mais importante da página.
   Galeria visual (fotos, vídeos, cases) que é a prova concreta do
   trabalho. Deve ocupar mais espaço que qualquer outra seção.
4. **Prova social** — depoimentos, marcas/clientes já atendidos,
   números relevantes (anos de experiência, projetos, seguidores),
   apresentados de forma sutil, não como "certificados de vendas".
5. **Diferenciais / forma de trabalho** — o que torna essa pessoa
   diferente, como ela atua (não é uma lista de "benefícios do
   produto").
6. **Contato** — call-to-action único e discreto: agendar, chamar no
   WhatsApp, seguir no Instagram. Sem senso de urgência artificial
   ("últimas vagas", "oferta por tempo limitado") — o objetivo é
   parecer seletivo e desejável, não promocional.
7. **FAQ** — só incluir se o nicho tiver dúvidas recorrentes reais
   sobre processo/agendamento (ex: "como funciona a consulta",
   "atende online?"). Nunca objeções de venda ("por que devo
   comprar?").

As seções 4 e 5 acima mapeiam pra `CampaignsSection.tsx` e
`ProcessSection.tsx` (ver "O motor comprovado"), mas os blocos internos
delas (chips de categoria, carrossel de marcas parceiras) são **opcionais
e adaptáveis por nicho** — ex: "Marcas Parceiras" faz sentido pra
modelo/influencer, mas pra um personal trainer ou nutricionista pode virar
"Parceiros & Certificações" ou simplesmente não existir. Não force uma
seção que não se aplica ao nicho do cliente.

Regras de CTA:
- Um único CTA principal, repetido no máximo 2 vezes (abertura e
  contato final) — nunca insistente.
- Linguagem de convite/seleção, não de venda: "Vamos conversar",
  "Agende uma consulta", "Fale comigo" — evitar "compre agora",
  "garanta sua vaga", "aproveite".

## Tom de copy por nicho

- **Nutricionista / personal trainer**: tom próximo e confiável,
  focado em resultado real e método de trabalho. Menos aspiracional,
  mais direto e humano.
- **Modelo comercial** (buscando parcerias com marcas/provadores):
  tom editorial, visual em primeiro lugar — o texto é mínimo, quase
  todo o peso está nas imagens/vídeos do portfólio.
- **Tatuador**: tom autoral e artístico, portfólio como galeria de
  arte (grid de trabalhos), estética mais crua/dark, tipografia com
  personalidade.
- **Influencer**: tom aspiracional, mistura de portfólio de conteúdo
  com número/prova de audiência, mas sem parecer "mídia kit"
  corporativo.

Em todos os casos: a página deve parecer curada e exclusiva, nunca
genérica ou "gerada em massa" — mesmo vindo do mesmo template.

## Regras de design

- Paleta de cores: extrair da identidade visual do cliente (prints,
  logo, fotos), usando as "Direções de partida" acima só como ponto de
  partida de humor visual. Nunca usar paleta genérica "roxo com
  gradiente" por padrão.
- Tipografia: uma fonte serifada ou de destaque para títulos (reforça
  o tom premium/editorial), uma fonte sans-serif limpa para corpo de
  texto.
- Imagens/vídeos em alta qualidade são o ativo mais importante da
  página — nunca comprimir a ponto de perder nitidez, e priorizar
  layout que dê espaço grande para elas (galeria full-bleed, grid
  editorial), evitando thumbnails pequenos.
- Toda foto/vídeo do cliente deve ter uma versão **portrait** (retrato,
  9:16, usada no mobile) e, quando o cliente também fornecer, uma versão
  **landscape** (paisagem, 16:9, usada só no desktop via `desktopSrc` do
  `FullBleedMedia` — ver "O motor comprovado"). Se só existir a portrait,
  ela é usada em todas as telas — nunca bloquear a página esperando a
  landscape. Nomear os arquivos como `[Nome] NN - PT.ext` / `[Nome] NN -
  LD.ext` (ex: `Isabella 01 - PT.mp4` / `Isabella 01 - LD.mp4`), guardados
  em `assets/images/portrait|landscape/` e
  `assets/videos/portrait|landscape/` (ver "Estrutura de pastas").
- Mobile-first: testar sempre a partir de 375px de largura antes de
  ajustar para desktop.
- Espaçamento generoso entre seções (mínimo 80px em desktop, 48px em
  mobile) — o respiro visual reforça a sensação de exclusividade.
- Tipografia fluida (`clamp()` com `vw`): sempre testar as duas pontas —
  celular pequeno **e** desktop grande — antes de considerar fechado.
  `vw` sem um teto reduzido pra telas largas faz o texto "estourar" no
  desktop mesmo funcionando bem no mobile.
- Degradê escuro sobre foto clara sempre aparece como linha/mancha
  visível, não importa o tamanho ou a opacidade do degradê — é um
  problema de contraste de cor, não de dimensão. Se a mídia de fundo for
  clara (comum em nichos como personal trainer/nutricionista — fotos de
  academia, cozinha, ambientes externos), resolver o contraste de texto
  pela seção inteira ter seu próprio overlay (o padrão já comprovado, ver
  "O motor comprovado") em vez de tentar disfarçar com um degradê fino
  isolado.

## Experiência de scroll

Este NÃO é um site com seções que só aparecem com fade-in ao entrar na
tela (isso parece estático e genérico). O conteúdo se revela
progressivamente conforme a rolagem, mas a mecânica comprovada (ver "O
motor comprovado" acima) é diferente do escrub contínuo estilo Apple
originalmente imaginado pra este projeto:

- **Transição entre seções**: CSS `scroll-snap` nativo — cada seção é uma
  "parada" (`snap-start snap-always`) do único container de scroll da
  página. Uma rolagem avança exatamente uma seção.
- **Reveal de conteúdo dentro da seção ativa**: `IntersectionObserver`
  detecta a seção visível e dispara um fade/translate com duração fixa
  via CSS — não escrubado pelo dedo/mouse.
- **Efeitos contínuos dentro de uma seção já ativa** (parallax leve de
  imagem de fundo, ken-burns/zoom, contagem progressiva de número): aí
  sim usar GSAP com `scrub: true` ou `scrub: 1`, ou uma timeline
  disparada pelo `IntersectionObserver`.
- **Zoom lento (ken-burns) padrão em toda seção de imagem cheia**: escala
  de 1 para ~1.08 ao longo de ~6s (`ease: 'sine.out'`) enquanto a seção
  está ativa, voltando pra 1 em 0.6s (`power2.out`) ao sair — disparado
  por `IntersectionObserver`, não GSAP `scrub`. É o comportamento default
  pra toda imagem full-bleed (Portfólio, Processo, Contato); não se
  aplica a vídeo (que já tem movimento próprio) nem a uma seção que já
  tenha um efeito contínuo diferente e intencional (ex: o parallax
  vertical leve do Sobre).
- **Hero em vídeo**: a primeira seção sempre tem um vídeo em loop, mudo,
  autoplay, cobrindo toda a tela como fundo. O texto (nome,
  posicionamento) fica sobreposto na frente do vídeo, nunca abaixo dele —
  overlay escuro sutil (gradiente) atrás do texto só o suficiente pra
  garantir contraste, sem esconder o vídeo. Anima com GSAP no primeiro
  load e reanima (`replayOnScroll`) toda vez que a seção volta a ficar
  visível.
- **Seção de portfólio/trabalho**: cada item da galeria é uma seção cheia
  própria dentro do mesmo scroll-snap — não `ScrollTrigger.pin` com
  scroller isolado (ver "por quê" em "O motor comprovado").
- **Performance**: vídeos de fundo devem ser comprimidos e leves
  (idealmente MP4 H.264, poucos segundos em loop) — nunca sacrificar o
  scroll suave por causa de vídeo pesado.
- Repetir esse padrão (mídia de fundo + texto revelado ao ficar ativo) em
  toda seção com foto/vídeo grande — Hero, Sobre, Portfólio, Processo e
  Contato usam a mesma lógica.

## Regras de código

- Componentes em TypeScript, tipados (evitar `any`).
- Um componente por arquivo, nomes em PascalCase.
- Dados do cliente num único arquivo tipado em `src/content/` (ex:
  `content.ts` ou o nome do cliente), espelhando a forma já usada no
  arquivo de referência deste projeto (`src/content/isabella.ts`): nome,
  bio curta/longa, stats, categorias, stats sociais, itens de portfólio
  com legenda/descrição, links de contato/Instagram. Renomear esse
  arquivo por cliente — não deixar `isabella` num projeto novo.
- Sem bibliotecas de CSS além do Tailwind, salvo necessidade
  específica.

## O que NÃO fazer

- Não estruturar a página como landing page de vendas (sem "oferta",
  "benefícios do produto", contadores de urgência, múltiplos CTAs
  insistentes).
- Não usar templates genéricos sem adaptar à identidade visual e ao
  tom do nicho do cliente.
- Não reutilizar a mesma paleta/composição visual entre clientes de
  nichos diferentes — cada página deve parecer feita sob medida.
- Não fechar um projeto novo numa única proposta de cara — gerar no
  mínimo 2 páginas (ver "Múltiplas propostas por cliente"), salvo pedido
  explícito do cliente pra já seguir uma direção única.
- Não deixar a Home seletora de direções nem rotas de comparação no
  projeto final de um cliente, salvo pedido explícito de comparação.
- Não inventar/usar URL de imagem ou vídeo de banco externo em modo
  placeholder (ver "Modo sem assets") — usar bloco de cor sólida/gradiente.
- Não publicar sem revisão manual de responsividade, qualidade de
  imagem/vídeo e acessibilidade básica (contraste de texto, tamanho de
  fonte mínimo).

---
> Source: [doupsdigital/projeto-personal](https://github.com/doupsdigital/projeto-personal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
